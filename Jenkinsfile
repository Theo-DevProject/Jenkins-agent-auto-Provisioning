pipeline {
  agent { label 'agent-amz' } // EC2 plugin launches Amazon Linux agent per run

  environment {
    S3_BUCKET = ''                  // optional, e.g., 'my-private-bucket'
    ARTIFACT  = 'log_stats_bundle.zip'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Package Artifact') {
      steps {
        sh '''
          rm -f ${ARTIFACT}
          zip -r ${ARTIFACT} ansible init_db.sql log_stats.py requirements.txt
        '''
        archiveArtifacts artifacts: "${ARTIFACT}", fingerprint: true
      }
    }

    stage('Upload to S3 (optional)') {
      when { expression { return env.S3_BUCKET?.trim() } }
      steps {
        sh 'aws s3 cp ${ARTIFACT} s3://${S3_BUCKET}/artifacts/${ARTIFACT}'
      }
    }

    stage('Deploy via Ansible') {
      steps {
        sh 'ansible-playbook -i ansible/inventories/prod/hosts.ini ansible/deploy_log_stats.yml'
      }
    }
  }

  post {
    always { cleanWs() }
  }
}
