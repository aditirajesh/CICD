pipeline {
  agent any
  environment { TF_DIR = "day1" }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Init') {
      steps { dir(env.TF_DIR) { sh 'terraform init -input=false' } }
    }

    stage('Plan') {
      steps { dir(env.TF_DIR) { sh 'terraform plan -input=false -out=tfplan' } }
    }

    stage('Approval') {
      when {
        expression {
          def b = (env.GIT_BRANCH ?: "").trim()
          if (b) {
            return b == 'origin/main' || b == 'origin/master' || b == 'main' || b == 'master'
          }
          def inferred = sh(script: "git name-rev --name-only --refs=refs/remotes/origin/* HEAD | head -n 1", returnStdout: true).trim()
          return inferred.contains("origin/main") || inferred.contains("origin/master")
        }
      }
      steps {
        input message: "Approve Terraform apply?", ok: "Apply"
      }
    }

    stage('Apply') {
      when {
        expression {
          def b = (env.GIT_BRANCH ?: "").trim()
          if (b) {
            return b == 'origin/main' || b == 'origin/master' || b == 'main' || b == 'master'
          }
          def inferred = sh(script: "git name-rev --name-only --refs=refs/remotes/origin/* HEAD | head -n 1", returnStdout: true).trim()
          return inferred.contains("origin/main") || inferred.contains("origin/master")
        }
      }
      steps {
        dir(env.TF_DIR) { sh 'terraform apply -input=false -auto-approve tfplan' }
      }
    }
  }
}
