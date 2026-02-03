pipeline {
    agent any

    options {
        timestamps()
    }

    parameters {
        choice(
            name: 'RELEASE_TYPE',
            choices: ['none', 'patch', 'minor', 'major'],
            description: 'Select release type for Ansible role'
        )
    }

    environment {
        ROLE_NAME = "nginx"
    }

    stages {

        /* ---------------- CHECKOUT ---------------- */

        stage('Checkout Role Code') {
            steps {
                checkout scm
            }
        }

        /* ---------------- CI VALIDATION ---------------- */

        stage('Role CI Validation') {
            steps {
                echo "Running Ansible syntax check"
                sh 'ansible-playbook --syntax-check roles/nginx/tests/test.yml'

                echo "Running ansible-lint"
                sh 'ansible-lint roles/nginx'
            }
        }

        /* ---------------- RELEASE TRIGGER ---------------- */

        stage('Release Triggered') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "Release triggered for role '${ROLE_NAME}'"
            }
        }

        /* ---------------- APPROVAL ---------------- */

        stage('Approval') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                input message: "Approve ${params.RELEASE_TYPE} release for role '${ROLE_NAME}'?",
                      ok: 'Approve Release'
            }
        }

        /* ---------------- VERSION CALCULATION ---------------- */

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

                    def (v, major, minor, patch) = lastTag.tokenize('.')
                    major = major.replace('v','').toInteger()
                    minor = minor.toInteger()
                    patch = patch.toInteger()

                    if (params.RELEASE_TYPE == 'patch') {
                        patch++
                    } else if (params.RELEASE_TYPE == 'minor') {
                        minor++
                        patch = 0
                    } else if (params.RELEASE_TYPE == 'major') {
                        major++
                        minor = 0
                        patch = 0
                    }

                    env.NEW_VERSION = "v${major}.${minor}.${patch}"
                    echo "Calculated version: ${env.NEW_VERSION}"
                }
            }
        }

        /* ---------------- TAG ROLE ARTIFACT ---------------- */

        stage('Tag Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-pat',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    sh '''
                      git config user.name "hardbro786"
                      git config user.email "modihardik19@gmail.com"

                      git tag ${NEW_VERSION}

                      git push https://${GIT_USER}:${GIT_TOKEN}@github.com/hardbro786/ansible-ci.git ${NEW_VERSION}
                    '''
                }
            }
        }

        /* ---------------- PUBLISH ARTIFACT ---------------- */

        stage('Publish Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "Role '${ROLE_NAME}' released as ${NEW_VERSION}"
                echo "Artifact is the Git tag itself"
            }
        }
    }

    post {
        success {
            echo "✅ Role CD completed successfully"
            echo "📦 Released ${ROLE_NAME}:${NEW_VERSION}"
        }
        failure {
            echo "❌ Role CD failed"
        }
        aborted {
            echo "⏹️ Role CD aborted"
        }
    }
}
