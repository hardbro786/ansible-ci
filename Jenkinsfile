pipeline {
    agent any

    parameters {
        choice(
            name: 'RELEASE_TYPE',
            choices: ['none', 'patch', 'minor', 'major'],
            description: 'Select role release type'
        )
    }

    environment {
        ROLE_NAME = "nginx"
        ROLE_PATH = "roles/nginx"
    }

    stages {

        stage('Checkout Role Code') {
            steps {
                checkout scm
            }
        }

        stage('Role CI Validation') {
            steps {
                echo "Running syntax check on role"
                sh """
                  ansible-playbook --syntax-check ${ROLE_PATH}/tests/test.yml
                """

                echo "Running ansible-lint"
                sh """
                  ansible-lint ${ROLE_PATH}
                """
            }
        }

        stage('Release Triggered') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "Release triggered for role: ${ROLE_NAME}"
            }
        }

        stage('Approval') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                input message: "Approve ${params.RELEASE_TYPE} release for role '${ROLE_NAME}'?",
                      ok: 'Approve Release'
            }
        }

        stage('Version Calculation') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                script {
                    def lastTag = sh(
                        script: "git describe --tags --abbrev=0 || echo v0.0.0",
                        returnStdout: true
                    ).trim()

                    def v = lastTag.replace('v','').split('\\.')
                    int major = v[0] as int
                    int minor = v[1] as int
                    int patch = v[2] as int

                    if (params.RELEASE_TYPE == 'major') {
                        major++; minor = 0; patch = 0
                    } else if (params.RELEASE_TYPE == 'minor') {
                        minor++; patch = 0
                    } else if (params.RELEASE_TYPE == 'patch') {
                        patch++
                    }

                    env.NEW_VERSION = "v${major}.${minor}.${patch}"
                    echo "New role version: ${env.NEW_VERSION}"
                }
            }
        }

        stage('Tag Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                sh """
                  git tag ${NEW_VERSION}
                  git push origin ${NEW_VERSION}
                """
            }
        }

        stage('Publish Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "Role ${ROLE_NAME}:${NEW_VERSION} is now approved and published"
                echo "Can be consumed by Playbook CD"
            }
        }
    }

    post {
        success {
            echo "✅ Role CD completed successfully"
            echo "✅ Current approved role: ${ROLE_NAME}:${NEW_VERSION}"
        }
        failure {
            echo "❌ Role CD failed"
        }
        aborted {
            echo "⏸ Release aborted by user"
        }
    }
}
