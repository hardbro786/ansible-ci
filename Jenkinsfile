pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'RELEASE',
            defaultValue: false,
            description: 'Trigger role release'
        )
    }

    environment {
        ROLE_NAME = "my_role"
        VERSION   = "v2.0.0"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/hardbro786/ansible-ci.git',
                    branch: 'main'
            }
        }

        stage('Role (CI Checks Passed)') {
            steps {
                echo 'Running ansible-lint'
                sh 'ansible-lint roles/my_role'

                echo 'Running molecule tests'
                sh 'molecule test'
            }
        }

        stage('Release Triggered') {
            when {
                expression { params.RELEASE == true }
            }
            steps {
                echo 'Release has been triggered'
            }
        }

        stage('Approval') {
            when {
                expression { params.RELEASE == true }
            }
            steps {
                input message: "Approve release of role ${ROLE_NAME}?",
                      ok: 'Approve'
            }
        }

        stage('Tag Created') {
            when {
                expression { params.RELEASE == true }
            }
            steps {
                sh """
                  git tag ${VERSION}
                  git push origin ${VERSION}
                """
            }
        }
    }

    post {
        success {
            echo "Current Tagged Artifact Ready: ${ROLE_NAME}:${VERSION}"
        }
        aborted {
            echo "Release aborted during approval stage"
        }
        failure {
            echo "Role CI or release failed"
        }
    }
}
