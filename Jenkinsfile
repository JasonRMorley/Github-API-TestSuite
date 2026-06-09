pipeline {
    agent any

    options {
        skipDefaultCheckout()
        timestamps()
    }

    environment {
        COLLECTION_FILE = 'Postman/github-api.postman_collection.json'
        ENVIRONMENT_FILE = 'Postman/github-api-environment.template.json'
        REPORT_FILE = 'newman-report.html'

        BASE_URL = 'https://api.github.com'
        GITHUB_USERNAME = 'JasonRMorley'
        TEST_REPO_NAME = 'newman-test-repo'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/JasonRMorley/Github-API-TestSuite.git'
            }
        }

        stage('Verify Newman Installed') {
            steps {
                sh '''
                    node --version
                    npm --version
                    newman --version
                    npm list -g newman-reporter-htmlextra
                '''
            }
        }

        stage('Run Newman Tests') {
            steps {
                withCredentials([
                    string(credentialsId: 'github-fine-token', variable: 'GITHUB_FINE_TOKEN')
                ]) {
                    sh '''
                        newman run "$COLLECTION_FILE" \
                            -e "$ENVIRONMENT_FILE" \
                            --env-var "github-fine-token=$GITHUB_FINE_TOKEN" \
                            --env-var "base_url=$BASE_URL" \
                            --env-var "repoOwner=$GITHUB_USERNAME" \
                            --env-var "repoName=$TEST_REPO_NAME" \
                            -r cli,htmlextra \
                            --reporter-htmlextra-export "$REPORT_FILE"
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'newman-report.html', fingerprint: true, allowEmptyArchive: true

            publishHTML(target: [
                allowMissing: true,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'newman-report.html',
                reportName: 'Newman Test Report'
            ])
        }
    }
}
