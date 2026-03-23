pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'git status'
                sh 'git pull origin main'
            }
        }
        stage('Test') {
            steps {
                sh 'git log -1'
            }
        }
    }
}
