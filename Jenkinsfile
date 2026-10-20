pipeline {
    agent any
    stages {
        stage('run') {
            steps {
                echo 'Welcome to Jenkins'
                sh 'python3 --version'
                sh 'python3 pipeline.py'
            }
        }
    }
}
