pipeline {
  agent {
    docker { image 'python:3.12-slim' }
  }
  environment {
    HOME = "${WORKSPACE}"                 // clave: evita que pip use /.local
    PIP_CACHE_DIR = "${WORKSPACE}/.pip-cache"
    PIP_DISABLE_PIP_VERSION_CHECK = "1"
  }
  stages {
    stage('Security gate - Bandit (HIGH)') {
      steps {
        sh '''
          python --version
          python -m pip --version

          # No hace falta "pip -U pip"
          python -m pip install --user --no-cache-dir bandit

          # Gate: solo severidad HIGH (y opcionalmente confianza HIGH)
          python -m bandit -r . --severity-level high --confidence-level high -f json -o bandit.json
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'bandit.json', fingerprint: true
        }
      }
    }
  }
}

