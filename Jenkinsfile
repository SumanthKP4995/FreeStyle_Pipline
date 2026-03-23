pipeline {
  agent any

  // ------------- Global Config -------------
  options {
    timestamps()                   // Timestamp every log line
    disableConcurrentBuilds()      // Prevent overlapping runs
    timeout(time: 30, unit: 'MINUTES')
  }

  parameters {
    choice(name: 'DEPLOY_ENV', choices: ['dev', 'qa', 'prod'], description: 'Where to deploy?')
    booleanParam(name: 'RUN_SECURITY_SCAN', defaultValue: true, description: 'Run container security scan?')
  }

  environment {
    APP_NAME      = 'sample-app'
    DOCKER_REG    = 'registry.example.com'
    DOCKER_IMG    = "${env.DOCKER_REG}/${env.APP_NAME}"
    IMAGE_TAG     = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    // Example: Maven local repo caching per workspace
    MAVEN_OPTS    = '-Dmaven.test.failure.ignore=true'
  }

  tools {
    jdk   'jdk-17'        // Configure in Manage Jenkins > Global Tool Configuration
    maven 'maven-3.9.6'   // Same as above
  }

  // ------------- Stages -------------
  stages {

    stage('Checkout') {
      steps {
        echo "Branch: ${env.BRANCH_NAME}, Build: #${env.BUILD_NUMBER}"
        checkout scm
        // Save source for later parallel stages
        stash name: 'src', includes: '**/*', excludes: '.git/**, target/**'
      }
    }

    stage('Build') {
      steps {
        unstash 'src'
        sh 'mvn -B -U -q clean compile'
        // Save compiled classes to speed up test stages
        stash name: 'build-output', includes: 'target/**/*', excludes: 'target/surefire-reports/**'
      }
    }

    stage('Quality & Tests (Parallel)') {
      parallel {
        stage('Unit Tests') {
          agent { label 'linux' }
          steps {
            unstash 'src'
            sh 'mvn -q -DskipITs=true test'
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
          }
        }

        stage('Integration Tests') {
          agent { label 'linux' }
          steps {
            unstash 'src'
            // Example profile for ITs; adjust to your project
            sh 'mvn -q -P integration-tests -DskipITs=false verify'
          }
          post {
            always {
              junit 'target/failsafe-reports/*.xml'
            }
          }
        }

        stage('Static Analysis') {
          agent { label 'linux' }
          steps {
            unstash 'src'
            // Replace with your analyzer; e.g., SpotBugs/Checkstyle
            sh 'mvn -q -DskipTests=true spotbugs:spotbugs checkstyle:checkstyle || true'
          }
          post {
            always {
              // Archive reports if present
              archiveArtifacts allowEmptyArchive: true, artifacts: 'target/spotbugsXml.xml, target/checkstyle-result.xml'
            }
          }
        }
      }
    }

    stage('Package') {
      steps {
        unstash 'src'
        // Build artifact(s) to be containerized/deployed
        sh 'mvn -B -q -DskipTests package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Build & Push Image') {
      when { not { branch 'feature/*' } }
      environment {
        // Jenkins credential id: docker-registry-cred (username/password or token)
        DOCKER_CREDS = credentials('docker-registry-cred')
      }
      steps {
        script {
          sh """
            echo "${DOCKER_CREDS_PSW}" | docker login ${DOCKER_REG} -u "${DOCKER_CREDS_USR}" --password-stdin
            docker build -t ${DOCKER_IMG}:${IMAGE_TAG} .
            docker tag ${DOCKER_IMG}:${IMAGE_TAG} ${DOCKER_IMG}:latest
            docker push ${DOCKER_IMG}:${IMAGE_TAG}
            docker push ${DOCKER_IMG}:latest
          """
        }
      }
      post {
        success {
          echo "Pushed image: ${DOCKER_IMG}:${IMAGE_TAG}"
        }
        cleanup {
          sh """
            docker rmi ${DOCKER_IMG}:${IMAGE_TAG} || true
            docker rmi ${DOCKER_IMG}:latest || true
          """
        }
      }
    }

    stage('Security Scan') {
      when { expression { return params.RUN_SECURITY_SCAN } }
      steps {
        // Replace with your scanner (e.g., Trivy, Grype). Example with Trivy:
        sh """
          which trivy >/dev/null 2>&1 || echo 'NOTE: Install Trivy on agent to enable scanning.'
          trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMG}:${IMAGE_TAG} || true
        """
      }
    }

    stage('Deploy') {
      when {
        anyOf {
          branch 'main'
          branch pattern: 'release/.*', comparator: 'REGEXP'
        }
      }
      steps {
        script {
          if (params.DEPLOY_ENV == 'prod') {
            // Manual approval gate for prod
            input message: "Deploy ${APP_NAME} to PROD with image ${IMAGE_TAG}?", ok: 'Deploy'
          }
        }
        // Example: call your deploy script/playbook/helm chart
        sh """
          echo "Deploying ${APP_NAME}:${IMAGE_TAG} to ${params.DEPLOY_ENV}"
          ./scripts/deploy.sh ${params.DEPLOY_ENV} ${DOCKER_IMG}:${IMAGE_TAG}
        """
      }
    }
  }

  // ------------- Post Actions -------------
  post {
    success {
      echo "✅ Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    unstable {
      echo "🟡 Unstable: Check tests/quality reports."
    }
    failure {
      echo "❌ Failed: Investigate logs and reports."
    }
    always {
      // Keep key logs & reports
      archiveArtifacts allowEmptyArchive: true, artifacts: 'target/site/**, target/*.log'
      // Example: lightweight notification (replace with Slack/Email steps)
      echo "Build URL: ${env.BUILD_URL}"
    }
  }
}
