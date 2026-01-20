pipeline {
  agent any

  stages {

    stage('Checkout') {
      steps {
        git 'https://github.com/org/ansible-roles.git'
      }
    }

    stage('Syntax Check') {
      steps {
        sh 'ansible-playbook roles/nginx/tests/test.yml -i roles/nginx/tests/inventory --syntax-check'
      }
    }

    stage('Lint') {
      steps {
        sh 'ansible-lint roles/nginx'
      }
    }

    stage('Dry Run') {
      steps {
        sh 'ansible-playbook roles/nginx/tests/test.yml -i roles/nginx/tests/inventory --check'
      }
    }
  }

  post {
    success {
      echo " CI Passed – Role is Stable"
    }
    failure {
      echo " CI Failed – Fix issues before merge"
    }
  }
}
