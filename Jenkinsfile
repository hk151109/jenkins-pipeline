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

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        bat 'echo Node version: && node -v'
        bat 'echo NPM version: && npm -v'
        bat 'npm ci --silent'
      }
    }

    stage('Test') {
      steps {
        // Pass even if there are no tests; keep output minimal
        bat 'npm test -- --watchAll=false --passWithNoTests --silent'
      }
    }

    stage('Build') {
      steps {
        bat 'npm run build --silent'
      }
      post {
        success {
          // Keep the built assets as artifacts
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
          xcopy build "%DEPLOY_DIR%" /E /I /Y >nul
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
      echo "[PIPELINE] ❌ Failed: Check stage logs above."
    }
    always {
      echo "[PIPELINE] Finished with status: ${currentBuild.currentResult}"
    }
  }
}
