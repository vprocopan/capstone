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
            }
        }
        stage('Deploy') { 
            steps {
                echo  'Deploying..'
            }
        }
    }
}