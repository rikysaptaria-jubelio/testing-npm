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

        stage('Audit') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }
    }

    post {
        success {
            echo '✅ Aman'
        }
        failure {
            echo '🚨 Ada vulnerability!'
        }
    }
}
