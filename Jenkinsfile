pipeline {

  agent any

  environment {

    // ===== JFrog Config =====
    JFROG_REGISTRY = 'trial1vvj9n.jfrog.io'
    JFROG_REPOSITORY = 'docker-local-docker'

    IMAGE_NAME = 'my-nginx'
    IMAGE_TAG = "${BUILD_NUMBER}"

  }

  stages {

    // =========================================================
    // VALIDATION
    // =========================================================

    stage('Validation') {

      parallel {

        stage('Jira Validation') {

          steps {

            script {

              if (env.BRANCH_NAME == 'main') {
                echo "Skipping Jira validation for main"
                return
              }

              if (!(env.BRANCH_NAME ==~ /.*[A-Z]+-[0-9]+.*/)) {
                error "Jira ID missing in branch name (e.g., ABC-123)"
              }

              echo "Jira validation passed"
            }
          }
        }

        stage('Milestone Validation') {

          steps {

            script {

              if (env.BRANCH_NAME.startsWith("release/")) {
                echo "Release branch detected"
              } else {
                echo "Normal development branch"
              }

            }
          }
        }

      }
    }

    // =========================================================
    // CHECKOUT
    // =========================================================

    stage('Checkout') {

      steps {
        checkout scm
      }
    }

    // =========================================================
    // INSTALL
    // =========================================================

    stage('Install') {

      steps {
        sh 'npm install'
      }
    }

    // =========================================================
    // TEST
    // =========================================================

    stage('Test') {

      steps {
        sh 'npm test -- --coverage'
      }
    }

    // =========================================================
    // COVERAGE
    // =========================================================

    stage('Coverage') {

      steps {

        recordCoverage(

          tools: [[
            parser: 'COBERTURA',
            pattern: 'coverage/cobertura-coverage.xml'
          ]],

          qualityGates: [[
            metric: 'LINE',
            threshold: 90
          ]]
        )
      }
    }

    // =========================================================
    // DOCKER BUILD
    // =========================================================

    stage('Docker Build') {

      steps {

        sh '''
          docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        '''
      }
    }

    // =========================================================
    // PUSH IMAGE TO JFROG
    // =========================================================

    stage('Push Image') {

      steps {

        withCredentials([usernamePassword(
          credentialsId: 'jfrog-creds',
          usernameVariable: 'JFROG_USER',
          passwordVariable: 'JFROG_PASS'
        )]) {

          sh '''

            echo "Logging into JFrog..."

            docker login ${JFROG_REGISTRY} \
              -u $JFROG_USER \
              -p $JFROG_PASS

            echo "Tagging Docker image..."

            docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
              ${JFROG_REGISTRY}/${JFROG_REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}

            echo "Pushing Docker image..."

            docker push \
              ${JFROG_REGISTRY}/${JFROG_REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}

          '''
        }
      }
    }

  }

  // =========================================================
  // POST ACTIONS
  // =========================================================

  post {

    success {

      echo "Pipeline Success"

    }

    failure {

      echo "Pipeline Failed"

    }

    always {

      archiveArtifacts(
        artifacts: 'coverage/**/*',
        allowEmptyArchive: true
      )

    }
  }
}