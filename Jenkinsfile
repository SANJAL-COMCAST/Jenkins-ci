pipeline {
  agent any

  stages {

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
        sh 'npm test -- --coverage'
      }
    }

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

  }

  post {

    success {
      echo "Pipeline Success"
    }

    failure {
      echo "Pipeline Failed"
    }

    always {
      archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
    }

  }
}