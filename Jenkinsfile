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
                sh '''
                    python3 -m venv .venv-ci
                    .venv-ci/bin/pip install --upgrade pip
                    .venv-ci/bin/pip install -r requirements.txt
                    .venv-ci/bin/pip install pip-audit
                    .venv-ci/bin/pip-audit -r requirements.txt
                '''
            }
        }

        stage('Django Check') {
            steps {
                sh '''
                    .venv-ci/bin/python manage.py check
                '''
            }
        }

        stage('SAST Scan') {
            steps {
                sh '''
                    .venv-ci/bin/pip install semgrep
                    .venv-ci/bin/semgrep scan --config auto .
                '''
            }
        }

        stage('Pipeline Success') {
            steps {
                echo 'Project4 pipeline stages completed'
            }
        }
    }
}
