pipeline {
  agent any
  stages {
    stage('Security gate - Bandit (HIGH)') {
      steps {
        sh '''
          python -m pip install -U pip
          python -m pip install bandit

          bandit -r . --severity-level high --confidence-level high -f json -o bandit.json
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
