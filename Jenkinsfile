pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code'
                
                git branch: 'main',
                    url: 'https://github.com/SumanthKP4995/FreeStyle_Pipline.git'

                sh 'git branch'
            }
        }

        stage('Build') {
            steps {
                echo 'Build stage - Git commands'
                sh '''
                  git status
                  git pull origin main
                  javac Main.java
                  java Main
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
