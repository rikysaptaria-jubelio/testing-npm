pipeline {
    agent any

    tools {
        nodejs 'node-lts'
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Audit JSON') {
            steps {
                sh 'npm audit --json > audit.json'
            }
        }

        stage('Analyze Result') {
            steps {
                script {
                    def audit = readJSON file: 'audit.json'

                    def high = audit.metadata.vulnerabilities.high
                    def critical = audit.metadata.vulnerabilities.critical
                    def moderate = audit.metadata.vulnerabilities.moderate

                    echo "Moderate: ${moderate}"
                    echo "High: ${high}"
                    echo "Critical: ${critical}"

                    // Logic
                    if (critical > 0 || high > 0) {
                        currentBuild.result = 'FAILURE'
                    } else if (moderate > 0) {
                        currentBuild.result = 'UNSTABLE'
                    } else {
                        currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }
    }

    post {
    failure {
        emailext(
            subject: "🚨 HIGH/CRITICAL Vulnerability Detected",
            body: "Check Jenkins build: ${env.BUILD_URL}",
            to: "itsec@jubelio.com"
        )
    }

    unstable {
        emailext(
            subject: "⚠️ Moderate Vulnerability Warning",
            body: "Check Jenkins build: ${env.BUILD_URL}",
            to: "itsec@jubelio.com"
        )
    }
}
}
