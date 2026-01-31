pipeline {
  agent {
    docker {
      image 'python:3.12-slim'
    }
  }
  environment {
    HOME = "${WORKSPACE}" // clave: evita que pip use /.local
    PIP_CACHE_DIR = "${WORKSPACE}/.pip-cache"
    PIP_DISABLE_PIP_VERSION_CHECK = "1"
    PROJECT_NAME = 'pygoat'
    PROJECT_VER = "build-${BUILD_NUMBER}"
    SBOM_FILE = 'bom.xml'
  }
  stages {
    /*stage('Security gate - Bandit (HIGH)') {
      steps {
        sh ''
        '
        python--version
        python - m pip--version

        # No hace falta "pip -U pip"
        python - m pip install--user--no - cache - dir bandit

        # Gate: solo severidad HIGH(y opcionalmente confianza HIGH)
        python - m bandit - r.--severity - level high--confidence - level high - f json - o bandit.json ''
        '
      }
      post {
        always {
          archiveArtifacts artifacts: 'bandit.json', fingerprint: true
        }
      }
    }*/
   stage('Generate SBOM') {
      steps {
        sh '''
          set -eux
          python3 -m venv .venv
          . .venv/bin/activate
    
          python -m pip install -U pip cyclonedx-bom
    
          # Opcional: si existe requirements.txt, instalar para que el SBOM refleje dependencias reales
          if [ -f requirements.txt ]; then
            python -m pip install -r requirements.txt
          fi
    
          # Genera SBOM desde el entorno del venv
          cyclonedx-py environment --of XML --sv 1.6 -o "${SBOM_FILE}"
          ls -lah "${SBOM_FILE}"
        '''
      }
    }
    
    stage('Upload to Dependency-Track') {
      steps {
        dependencyTrackPublisher(
          projectName: "${env.PROJECT_NAME}",
          projectVersion: "${env.PROJECT_VER}",
          artifact: "${env.SBOM_FILE}",
          synchronous: true
        )
      }
    }
  }
  post {
    always {
        archiveArtifacts artifacts: "${SBOM_FILE}", fingerprint: true
    }
  }
}
