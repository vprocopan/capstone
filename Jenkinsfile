pipeline {
  agent any

  environment {
    APP_NAME = "simple_nginx_app"
    AWS_ACCOUNT = "915323986442"
  }

  parameters {
    choice(name: 'DEP_VERSION', choices: ['Blue', 'Green'], description: 'Deployment Version')
  }
  
  stages {

    stage('Linting') {
      steps {
        parallel(
          htmllint: {sh 'tidy -q -e views/*.html'},
          eslint: {sh 'eslint *.js'},
          hadolint: {sh 'hadolint Dockerfile'}
        )
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $APP_NAME .'
        sh 'docker image ls -q $APP_NAME:latest'
      }
    }

    stage('Scan Docker Image') {
      steps {
        aquaMicroscanner imageName: 'simple_nxinx_app:latest', notCompliesCmd: 'exit 4', onDisallowed: 'fail', outputFormat: 'html'
      }
    }

    stage('Run and Test App in Docker') {
      steps {
        sh 'docker run --name $APP_NAME -p 80:80 -d $APP_NAME'
        sh 'sleep 5'
        sh 'curl -s http://localhost:80'
        sh 'docker logs $APP_NAME'
        sh 'docker stop $APP_NAME'
        sh 'docker rm $APP_NAME'
      }
    }

    stage('Push image to DockerHub') {
      steps {
        withDockerRegistry(credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/') {
          sh 'docker tag $APP_NAME softveda/$APP_NAME:$BUILD_NUMBER'
          sh 'docker tag $APP_NAME softveda/$APP_NAME:latest'
          sh 'docker push softveda/$APP_NAME:$BUILD_NUMBER'
          sh 'docker push softveda/$APP_NAME:latest'
        }
      }
      post {
        always {
          sh 'docker image rm -f softveda/$APP_NAME:$BUILD_NUMBER'
          sh 'docker image rm -f softveda/$APP_NAME:latest'
        }
      }
    }

    stage('Create ECR repository') {
      steps {
        sh 'aws cloudformation deploy --stack-name simple-node-app-repo --region us-west-2 --template-file cfn-ecr.yml --parameter-overrides RepositoryName=$APP_NAME'
      }
    }

    stage('Push image to ECR') {
      steps {
        sh 'docker tag $APP_NAME $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:$BUILD_NUMBER'
        sh 'docker push $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:$BUILD_NUMBER'
        sh 'docker tag $APP_NAME $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:latest'
        sh 'docker push $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:latest'        
      }
      post {
        always {
          sh 'docker image rm -f $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:$BUILD_NUMBER'
          sh 'docker image rm -f $AWS_ACCOUNT.dkr.ecr.us-west-2.amazonaws.com/$APP_NAME:latest'
          sh 'docker image rm -f $APP_NAME:latest'
        }
      }
    }

    stage('Deploy to EKS') {
      environment {
        EKS_STATUS = sh (script:'eksctl get cluster --name=basic-cluster --region=us-west-2', returnStatus: true)
      }
      when {
        expression {environment name: 'EKS_STATUS', value: '0'}
      }
      steps {
        echo "Deploying to EKS cluster ..."

        sh 'kubectl apply -f app-deployment-aws.yml'
        sh 'kubectl rollout status deployment.apps/simple-node-app --timeout=5m --watch=true'

        script {
          K8S_SVC = sh (
            script: "kubectl get services | grep simple-node-app | awk '{print \$4}'",
            returnStdout: true
          ).trim()
                
          echo "Kubernetes service URL: ${K8S_SVC}"
          sleep 5
          sh "curl -m 5 -S http://$K8S_SVC"
        }
      }
    }
  }
}