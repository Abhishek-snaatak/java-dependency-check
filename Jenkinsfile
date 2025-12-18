pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven-3'
    }

    environment {
        DC_REPORT_DIR = "${WORKSPACE}/dependency-check-report"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/OT-MICROSERVICES/salary-api'
            }
        }

        stage('Dependency Scan (OWASP)') {
            steps {
                sh """
                mkdir -p ${DC_REPORT_DIR}

                mvn -DskipTests \
                    org.owasp:dependency-check-maven:check \
                    -Dformat=ALL \
                    -DoutputDirectory=${DC_REPORT_DIR}
                """
            }
        }
    }

    post {

        success {
            script {
                def timestamp = sh(
                    script: "date +'%Y-%m-%d %H:%M:%S'",
                    returnStdout: true
                ).trim()

                def msg = """✅ BUILD SUCCESS
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Time (IST): ${timestamp}
• Report Path: ${DC_REPORT_DIR}
• Build URL: ${env.BUILD_URL}
"""

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    sh """
                    curl -X POST -H 'Content-Type: application/json' \
                    --data '{"text":"${msg}"}' \
                    "$SLACK_WEBHOOK_URL"
                    """
                }

                mail to: 'ps191701@gmail.com',
                     subject: "✅ Dependency Scan SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: msg
            }
        }

        failure {
            script {
                def timestamp = sh(
                    script: "date +'%Y-%m-%d %H:%M:%S'",
                    returnStdout: true
                ).trim()

                def msg = """❌ BUILD FAILURE
• Job: ${env.JOB_NAME}
• Build Number: #${env.BUILD_NUMBER}
• Time (IST): ${timestamp}
• Build URL: ${env.BUILD_URL}
"""

                withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_URL')]) {
                    sh """
                    curl -X POST -H 'Content-Type: application/json' \
                    --data '{"text":"${msg}"}' \
                    "$SLACK_WEBHOOK_URL"
                    """
                }

                mail to: 'ps191701@gmail.com',
                     subject: "❌ Dependency Scan FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: msg
            }
        }

        always {
            echo "Archiving OWASP Dependency Check Reports..."
            archiveArtifacts artifacts: 'dependency-check-report/**', allowEmptyArchive: true
        }
    }
}
