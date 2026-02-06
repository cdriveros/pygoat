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
    stage('Security gate - Bandit (HIGH)') {
      steps {
        sh ''
        '
        python--version
        python - m pip--version

        // Dependencias (pip): instalamos Bandit sin actualizar pip (no es necesario en el agente) y sin cache para builds más reproducibles y livianos. "--user" evita requerir permisos de root.
        python - m pip install--user--no - cache - dir bandit

        //Security Gate (Bandit): falla el build si detecta vulnerabilidades de severidad HIGH; opcionalmente filtra también por confianza HIGH para reducir falsos positivos. Genera reporte JSON en bandit.json para el paso de análisis/publicación.
        python - m bandit - r.--severity - level high--confidence - level high - f json - o bandit.json ''
        '
      }
      post {
        always {
          archiveArtifacts artifacts: 'bandit.json', fingerprint: true
        }
      }
    }
    stage('Generate SBOM') {
      steps {
        sh '''
          set -eux
          python3 -m venv .venv
          . .venv/bin/activate
    
          python -m pip install -U pip cyclonedx-bom
    
          # Genera el SBOM leyendo requirements.txt (sin instalar nada)
          if [ -f requirements.txt ]; then
            cyclonedx-py requirements requirements.txt --of XML --sv 1.6 -o "${SBOM_FILE}"
          else
            # fallback por si no existe requirements.txt
            cyclonedx-py environment --of XML --sv 1.6 -o "${SBOM_FILE}"
          fi
    
          ls -lah "${SBOM_FILE}"
        '''
      }
      post {
        always {
            archiveArtifacts artifacts: "${SBOM_FILE}", fingerprint: true
        }
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
    stage('Security gate - Gitleaks') { 
      agent { 
          docker { 
              image 'zricethezav/gitleaks:latest'
              args '--entrypoint ""'
              reuseNode true
          } 
      } 
      steps { 
          sh ''' 
              set -eux 
              # Ejecutar gitleaks dentro del contenedor (la imagen ya trae el binario) 
              # Escanea el workspace actual
              gitleaks detect \
                  --source . \
                  --report-format json \
                  --report-path gitleaks-report.json \
                  --redact \
                  --exit-code 1 
          '''
      } 
      post { 
          always { 
              archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true 
          }
      }
    }
  }
}
