pipeline {
    agent {
        docker {
            image 'node:18'
        }
    }

    stages {
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
