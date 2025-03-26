pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'echo Building...'
                // sh 'npm install'
                // sh 'npm run build'
            }
        }
    }
}
