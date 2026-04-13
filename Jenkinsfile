pipeline {
    agent any

    tools {
        nodejs 'node-lts'
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Prepare Lockfile (Safe)') {
            steps {
                sh '''
                if [ ! -f package-lock.json ]; then
                  npm install --ignore-scripts --package-lock-only
                fi
                '''
            }
        }

        stage('Audit JSON') {
            steps {
                sh 'npm audit --json > audit.json || true'
            }
        }

        stage('Analyze Result') {
            steps {
                script {
                    def audit = readJSON file: 'audit.json'

                    def high = audit.metadata.vulnerabilities.high ?: 0
                    def critical = audit.metadata.vulnerabilities.critical ?: 0
                    def moderate = audit.metadata.vulnerabilities.moderate ?: 0

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
                subject: "🚨 HIGH/CRITICAL Vulnerability",
                body: """\
Build: ${env.JOB_NAME}
Status: FAILURE
URL: ${env.BUILD_URL}

High/Critical vulnerabilities ditemukan.
""",
                to: "itsec@jubelio.com"
            )
        }

        unstable {
            emailext(
                subject: "⚠️ Moderate Vulnerability",
                body: """\
Build: ${env.JOB_NAME}
Status: UNSTABLE
URL: ${env.BUILD_URL}

Terdapat vulnerability level moderate.
""",
                to: "itsec@jubelio.com"
            )
        }

        success {
            echo '✅ Aman'
        }
    }
}
