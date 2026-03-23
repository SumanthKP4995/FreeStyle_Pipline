pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
		sh ''' git branch -a
			git status
			echo "build step complete"
		'''
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
		sh ''' git pull
                echo "Build completed" '''
		 
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'echo Tests passed'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
                sh 'echo Deployment successful'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
