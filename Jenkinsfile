pipeline {
  agent any

  stages {

    stage('Checkout') {
      steps {
        git 'https://github.com/hardbro786/ansible-ci.git'
	echo "Checkout Complete"
      }
    }

    stage('Syntax Check') {
      steps {
        sh 'ansible-playbook playbook.yml --syntax-check'
      }
    }

    stage('Lint') {
      steps {
        sh 'ansible-lint playbook.yml'
      }
    }

    stage('Dry Run') {
      steps {
        sh 'sudo ansible-playbook playbook.yml --check'
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
