pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'RELEASE_TYPE',
            choices: ['patch', 'minor', 'major', 'none'],
            description: 'Select release type'
        )
    }

    environment {
        ROLE_NAME = 'nginx'
        GIT_REPO  = 'github.com/hardbro786/ansible-ci.git'
        GIT_CRED  = 'github-pat'   // Jenkins credential ID
    }

    stages {

        stage('Checkout Role Code') {
            steps {
                checkout scm
            }
        }

        stage('Role CI Validation') {
            steps {
                echo "Running Ansible syntax check"
                sh 'ansible-playbook --syntax-check roles/nginx/tests/test.yml'

                echo "Running ansible-lint"
                sh 'ansible-lint roles/nginx'
            }
        }

        stage('Release Triggered') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "Release triggered for role '${ROLE_NAME}'"
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
                        script: 'git describe --tags --abbrev=0 || echo v0.0.0',
                        returnStdout: true
                    ).trim()

                    def (major, minor, patch) = lastTag.replace('v','').tokenize('.').collect { it as int }

                    if (params.RELEASE_TYPE == 'major') {
                        major++; minor = 0; patch = 0
                    } else if (params.RELEASE_TYPE == 'minor') {
                        minor++; patch = 0
                    } else {
                        patch++
                    }

                    env.NEW_VERSION = "v${major}.${minor}.${patch}"
                    echo "Calculated version: ${env.NEW_VERSION}"
                }
            }
        }

        stage('Tag Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${GIT_CRED}",
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    sh """
                      git config user.name "jenkins"
                      git config user.email "jenkins@company.com"

                      git tag ${NEW_VERSION}

                      git push https://${GIT_USER}:${GIT_TOKEN}@${GIT_REPO} ${NEW_VERSION}
                    """
                }
            }
        }

        stage('Publish Role Artifact') {
            when {
                expression { params.RELEASE_TYPE != 'none' }
            }
            steps {
                echo "✅ Role '${ROLE_NAME}' released as ${NEW_VERSION}"
                echo "📦 Artifact = Git tag (${NEW_VERSION})"
            }
        }
    }

    post {
        success {
            echo "🎉 Role CD completed successfully"
        }
        failure {
            echo "❌ Role CD failed"
        }
    }
}
