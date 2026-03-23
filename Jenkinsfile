pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code'
                
                sh 'git branch'
            }
        }

        stage('Build') {
            steps {
                echo 'Build stage - Git commands'
                sh '''
                  git status
                  git pull
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Test stage - Git commands'
                sh '''
                  git log -1
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploy stage - Git commands'
                sh '''
                  git branch
                  git show --oneline -1
                '''
            }
        }
    }
}
