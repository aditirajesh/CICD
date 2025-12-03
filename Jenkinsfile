def isMainBranch() {
  return sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim() in ['main','master']
}

pipeline {
  agent any

  environment {
    TF_DIR = "day1"
    // Optional: keep terraform non-interactive
    TF_IN_AUTOMATION = "true"
  }

  options {
    timestamps()
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Init') {
      steps {
        dir("${TF_DIR}") {
          sh '''
            terraform version
            terraform init -input=false
          '''
        }
      }
    }

    stage('Plan') {
      steps {
        dir("${TF_DIR}") {
          sh '''
            terraform fmt -check || true
            terraform validate
            terraform plan -input=false -out=tfplan
          '''
        }
      }
    }

    stage('Approval') {
      when { expression { isMainBranch() } }
      steps {
        input message: "Approve Terraform apply to create/update resources?", ok: "Apply"
      }
    }

    stage('Apply') {
      when { expression { isMainBranch() } }
      steps {
        dir("${TF_DIR}") {
          sh '''
            terraform apply -input=false -auto-approve tfplan
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Pipeline finished."
    }
  }
}
