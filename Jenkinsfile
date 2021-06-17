pipeline {
    agent any 
    stages {
        stage('Build') { 
            steps {
                sh ' echo  "Building.."'
                sh '''
                    ls -lah
                    '''
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
            }
        }
    }
}