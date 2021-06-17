pipeline {
    agent any 
    stages {
        stage('Build') { 
            steps {
                sh ' echo  "Building.."'
                sh '''
                    ls -lah
                    '''
                withDockerRegistry([url: "", credentialsId: "dockerhub-credentials"]){
                    sh 'docker build -t capstone-vprocopan .'
                    sh 'docker push vprocopan/capstone-vprocopan'

                }
                
            }
        }
        stage('Test') { 
            steps {
                echo  'Testing..'
                sh 'hadolint ./Dockerfile | tee -a hadolint_lint.txt'
            }
        }
        stage('Deploy') { 
            steps {
                echo  'Deploying..'
                dir('k8s') {
                    withAWS(credentials: 'aws-credentials', region: 'eu-west-2') {
                            sh "aws eks --region eu-west-2 update-kubeconfig --name capstone"
                            sh 'kubectl apply -f capstone-k8s.yaml'}
                        }
            }
        }
    }
}