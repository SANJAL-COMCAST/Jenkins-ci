pipeline {
  agent any

  stages {

    stage('Validation') {
      parallel {

        stage('Jira Validation') {
          steps {
            script {
              if (!env.BRANCH_NAME.matches(".*[A-Z]+-\\d+.*")) {
                error "Jira ID missing in branch name (e.g., ABC-123)"
              } else {
                echo "Jira validation passed"
              }
            }
          }
        }

        stage('Milestone Validation') {
          steps {
            script {
              if (!(env.BRANCH_NAME == 'main' || env.BRANCH_NAME.startsWith('release/') || env.BRANCH_NAME.startsWith('feature/'))) {
                error "Invalid branch for build"
              } else {
                echo "Milestone validation passed"
              }
            }
          }
        }

      }
    }

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
  }
}