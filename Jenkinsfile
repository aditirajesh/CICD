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
      when {
        anyOf {
          branch 'main'
          branch 'master'
        }
      }
      steps {
        input message: "Approve Terraform apply to create/update resources?", ok: "Apply"
      }
    }

    stage('Apply') {
      when {
        anyOf {
          branch 'main'
          branch 'master'
        }
      }
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
