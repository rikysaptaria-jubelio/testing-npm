pipeline {
    agent any

    tools {
        nodejs 'node-lts'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Audit') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }
    }

    post {
        success {
            echo '✅ Aman (no high vulnerabilities)'
        }
        failure {
            echo '🚨 Ada vulnerability!'
        }
    }
}
