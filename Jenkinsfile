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

                    def critical = 0
                    def high = 0
                    def moderate = 0

                    def details = ""

                    // 🔥 parsing vulnerability detail + hitung manual
                    audit.vulnerabilities.each { name, vuln ->
                        if (vuln.via instanceof List) {
                            vuln.via.each { item ->
                                if (item instanceof Map) {
                                    def severity = item.severity ?: "unknown"
                                    def title = item.title ?: "Unknown Issue"

                                    if (severity == 'critical') critical++
                                    else if (severity == 'high') high++
                                    else if (severity == 'moderate') moderate++

                                    details += "- ${name} → ${title} (${severity})\n"
                                }
                            }
                        }
                    }

                    // 🔹 Summary sesuai detail (FIX utama)
                    env.AUDIT_SUMMARY = """\
Summary:
- Critical: ${critical}
- High: ${high}
- Moderate: ${moderate}
"""

                    env.AUDIT_DETAILS = details ?: "No detailed vulnerabilities available"

                    // 🔥 Custom status
                    if (critical > 0 || high > 0 || moderate > 0) {
                        env.AUDIT_STATUS = "⚠️ Vulnerability Found"
                    } else {
                        env.AUDIT_STATUS = "✅ No Vulnerability Found"
                    }

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
                subject: "${env.AUDIT_STATUS} - ${env.JOB_NAME}",
                body: """\
Build: ${env.JOB_NAME}
Status: ${env.AUDIT_STATUS}
URL: ${env.BUILD_URL}

${env.AUDIT_SUMMARY}

Details:
${env.AUDIT_DETAILS}
""",
                to: "itsec@jubelio.com"
            )
        }
    }
}
