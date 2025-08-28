pipeline {
  agent any

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
      steps { checkout scm }
    }

    stage('Install') {
      steps {
        // Only print Node & npm versions
        bat 'echo Node version: && node -v'
        bat 'echo NPM version: && npm -v'
        // Run npm ci quietly (errors still show)
        bat 'npm ci --silent'
      }
    }

    stage('Test') {
      steps {
        // Quiet test run, ignore "no tests" with --passWithNoTests
        bat 'npm test -- --watchAll=false --passWithNoTests --silent'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'test-results/junit.xml'
        }
      }
    }

    stage('Build') {
      steps {
        // Build React app quietly
        bat 'npm run build --silent'
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
        bat '''
          if exist "%DEPLOY_DIR%" rmdir /S /Q "%DEPLOY_DIR%"
          mkdir "%DEPLOY_DIR%"
          robocopy build "%DEPLOY_DIR%" *.* /E >nul
          echo [DEPLOY] Local deploy done at %CD%\\%DEPLOY_DIR%
        '''
      }
    }
  }

  post {
    success {
      echo "[PIPELINE] ✅ Success: Build, Test, Deploy complete."
    }
    failure {
      echo "[PIPELINE] ❌ Failed: Check above logs."
    }
    always {
      echo "[PIPELINE] Finished with status: ${currentBuild.currentResult}"
    }
  }
}
