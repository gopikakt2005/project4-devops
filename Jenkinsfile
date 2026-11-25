pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Code checkout completed'
            }
        }

        stage('Dependency Scan') {
            steps {
                sh '''
                    docker run --rm \
                    -v "$WORKSPACE:/workspace" \
                    -w /workspace/project4 \
                    python:3.11-slim sh -c "
                        pip install -r requirements.txt
                        pip install pip-audit
                        pip-audit -r requirements.txt \
                        -f json -o dependency-report.json || true
                    "
                '''
            }
        }

        stage('Django Check') {
            steps {
                sh '''
                    docker run --rm \
                    -v "$WORKSPACE:/workspace" \
                    -w /workspace/project4 \
                    python:3.11-slim sh -c "
                        pip install -r requirements.txt
                        python manage.py check
                    "
                '''
            }
        }

        stage('SAST Scan') {
            steps {
                sh '''
                    docker run --rm \
                    -v "$WORKSPACE:/src" \
                    semgrep/semgrep \
                    semgrep scan --config auto /src/project4/ || true
                '''
            }
        }

        stage('Pipeline Success') {
            steps {
                echo 'All pipeline stages completed'
            }
        }
    }

    post {
        success {
            echo 'Project4 CI Pipeline SUCCESS!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
