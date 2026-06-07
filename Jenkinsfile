pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/JasonRMorley/Github-API-TestSuite.git'
            }
        }

        stage('Run Newman Tests') {
            steps {
                sh '''
                    newman run Postman/collections/repository_lifecycle.postman_collection \
                        -e Postman/environments/github_api-support-suite-env.postman_environment \
                        -r htmlextra \
                        --reporter-htmlextra-export newman-report.html
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'newman-report.html', fingerprint: true

            publishHTML(target: [
                allowMissing: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'newman-report.html',
                reportName: 'Newman Test Report'
            ])
        }
    }
}
