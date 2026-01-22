pipeline {
    agent any

    environment {
        ANSIBLE_ROLES_PATH = "${WORKSPACE}/roles"
        INVENTORY = "${WORKSPACE}/roles/nginx/tests/inventory"
        PLAYBOOK = "${WORKSPACE}/roles/nginx/tests/test.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/hardbro786/ansible-ci.git', branch: 'master'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y ansible python3-venv python3-pip
                    pip3 install --upgrade pip ansible-lint
                '''
            }
        }

        stage('Syntax Check') {
            steps {
                echo "✅ Running Ansible Syntax Check"
                sh "ansible-playbook ${PLAYBOOK} -i ${INVENTORY} --syntax-check"
            }
        }

        stage('Lint Roles') {
            steps {
                echo "✅ Running Ansible Lint"
                sh "ansible-lint ${ANSIBLE_ROLES_PATH}/nginx"
            }
        }

        stage('Dry Run / Check Mode') {
            steps {
                echo "✅ Running Ansible Dry Run (Check Mode)"
                sh "ansible-playbook ${PLAYBOOK} -i ${INVENTORY} --check"
            }
        }

        /* 🔐 MANUAL APPROVAL STAGE */
        stage('Approval for Deployment') {
            steps {
                input message: 'Do you want to deploy NGINX role to servers?',
                      ok: 'Approve Deployment'
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying Nginx Role to Target Hosts"
                sh "ansible-playbook ${PLAYBOOK} -i ${INVENTORY}"
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful – Role is Stable"
        }
        failure {
            echo "❌ Deployment Failed – Check Logs"
        }
        aborted {
            echo "⏸ Deployment Aborted – Approval Not Given"
        }
    }
}
