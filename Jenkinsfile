pipeline {
  agent any

  // No tools{} block — uses system Node already on PATH for the Jenkins service account
  environment {
    CI = 'true'
    DEPLOY_DIR = 'deploy'
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  triggers {
    pollSCM('@weekly')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        bat 'where node'
        bat 'node -v'
        bat 'npm -v'
        bat 'npm ci'
      }
    }

    stage('Test') {
      steps {
        bat 'npm test -- --watchAll=false'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/junit*.xml'
        }
      }
    }

    stage('Build') {
      steps {
        bat 'npm run build'
      }
      post {
        success {
          archiveArtifacts artifacts: 'build/**/*', fingerprint: true
        }
      }
    }

    stage('Deploy (local)') {
      when { expression { return fileExists('build') } }
      steps {
        // robust copy on Windows with robocopy
        bat '''
          if exist "%DEPLOY_DIR%" rmdir /S /Q "%DEPLOY_DIR%"
          mkdir "%DEPLOY_DIR%"
          robocopy build "%DEPLOY_DIR%" *.* /E >nul
          echo Local deploy done at %CD%\\%DEPLOY_DIR%
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished with status: ${currentBuild.currentResult}"
    }
  }
}
