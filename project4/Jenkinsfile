pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code checkout completed'
            }
        }

        stage('Dependency Scan') {
            steps {
                sh 'python3 -m pip install -r requirements.txt'
                sh 'python3 -m pip_audit -r requirements.txt'
            }
        }

        stage('Django Check') {
            steps {
                sh 'python3 manage.py check'
            }
        }

        stage('SAST Scan') {
            steps {
                sh 'semgrep scan --config auto .'
            }
        }

        stage('Pipeline Success') {
            steps {
                echo 'Project4 pipeline stages completed'
            }
        }
    }
}
