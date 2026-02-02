pipeline {
    agent any

    environment {
        ANSIBLE_ROLES_PATH = "${WORKSPACE}/roles"
        INVENTORY = "${WORKSPACE}/inventories/prod/hosts"
        PLAYBOOK = "${WORKSPACE}/playbooks/site.yml"
    }

    stages {

        stage('Checkout (Tag Based)') {
            steps {
                echo "Checking out tagged release: ${env.GIT_TAG}"
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'refs/tags/*']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/hardbro786/ansible-ci.git'
                    ]]
                ])
            }
        }

        stage('Approval') {
            steps {
                input message: "Approve deployment for tag ${env.GIT_TAG} ?",
                      ok: 'Approve Deployment'
            }
        }

        stage('Deploy via Ansible Controller') {
            steps {
                echo "Deploying using Ansible Controller"
                sh """
                    ansible-playbook ${PLAYBOOK} \
                    -i ${INVENTORY} \
                    --extra-vars "release_tag=${env.GIT_TAG}"
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful for tag ${env.GIT_TAG}"
        }
        aborted {
            echo "Deployment aborted – approval not granted"
        }
        failure {
            echo "Deployment failed – check logs"
        }
    }
}
