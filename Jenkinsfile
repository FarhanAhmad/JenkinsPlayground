pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo 'Compiling the Java source code'
                sh 'javac prag.java'
            }
        }
        stage('run') {
            steps {
                echo 'Running the compiled Java code.'
                sh 'java prag'
                sh 'python3 pipeline.py'
            }
        }
    }
}
