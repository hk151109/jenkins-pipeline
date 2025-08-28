pipeline {
  agent any

  tools {
    nodejs 'node18' 
  }

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
        sh 'node -v'
        sh 'npm -v'
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test -- --watchAll=false'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/junit*.xml'
        }
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
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
        sh '''
          rm -rf "${DEPLOY_DIR}"
          mkdir -p "${DEPLOY_DIR}"
          cp -r build/* "${DEPLOY_DIR}/"
          echo "Local deploy done at ${PWD}/${DEPLOY_DIR}"
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
