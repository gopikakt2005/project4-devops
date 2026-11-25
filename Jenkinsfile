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
                dir('project4') {
                    sh '''
                        python3 -m venv .venv-ci
                        .venv-ci/bin/pip install --upgrade pip
                        .venv-ci/bin/pip install -r requirements.txt
                        .venv-ci/bin/pip install pip-audit
                        .venv-ci/bin/pip-audit -r requirements.txt
                    '''
                }
            }
        }

        stage('Django Check') {
            steps {
                dir('project4') {
                    sh '.venv-ci/bin/python manage.py check'
                }
            }
        }

        stage('SAST Scan') {
            steps {
                sh '''
                    python3 -m venv .semgrep-venv
                    .semgrep-venv/bin/pip install semgrep
                    .semgrep-venv/bin/semgrep scan --config auto project4/
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
