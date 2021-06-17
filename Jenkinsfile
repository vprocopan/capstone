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
                sh 'hadolint ./Dockerfile'
            }
        }
        stage('Deploy') { 
            steps {
                echo  'Deploying..'
            }
        }
    }
}