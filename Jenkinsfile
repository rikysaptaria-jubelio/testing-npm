pipeline {
    agent any

    tools {
        nodejs 'node-lts'
    }

    options {
        timeout(time: 5, unit: 'MINUTES')
    }

    environment {
        AUDIT_STATUS = "UNKNOWN"
        AUDIT_SUMMARY = ""
        AUDIT_DETAILS = ""
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

                    // ===== STATUS =====
                    if (critical > 0 || high > 0 || moderate > 0) {
                        env.AUDIT_STATUS = "⚠️ Vulnerability Found"
                    } else {
                        env.AUDIT_STATUS = "✅ No Vulnerability Found"
                    }

                    // ===== SUMMARY =====
                    env.AUDIT_SUMMARY = """
- Critical: ${critical}
- High: ${high}
- Moderate: ${moderate}
"""

                    // ===== DETAILS (FIX NULL ISSUE) =====
                    def details = ""

                    audit.vulnerabilities.each { name, vuln ->
                        def via = vuln.via

                        if (via instanceof List && via.size() > 0) {
                            via.each { item ->
                                if (item instanceof Map) {
                                    def title = item.title ?: "Unknown Issue"
                                    def severity = item.severity ?: "unknown"

                                    details += "- ${name} → ${title} (${severity})\\n"
                                }
                            }
                        } else {
                            def severity = vuln.severity ?: "unknown"
                            details += "- ${name} → ${severity}\\n"
                        }
                    }

                    env.AUDIT_DETAILS = details

                    echo "=== AUDIT RESULT ==="
                    echo env.AUDIT_STATUS
                    echo env.AUDIT_SUMMARY
                    echo env.AUDIT_DETAILS
                }
            }
        }
    }

    post {
        always {
            emailext(
                subject: "[NPM AUDIT] ${env.AUDIT_STATUS}",
                body: """\
Build: ${env.JOB_NAME}
Status: ${env.AUDIT_STATUS}
URL: ${env.BUILD_URL}

Summary:
${env.AUDIT_SUMMARY}

Details:
${env.AUDIT_DETAILS}
""",
                to: "itsec@jubelio.com"
            )
        }

        success {
            echo '✅ Pipeline selesai'
        }
    }
}
