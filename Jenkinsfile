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
                    docker run --rm \
                    -v "$WORKSPACE:/workspace" \
                    -w /workspace/project4 \
                    python:3.11-slim sh -c '
                        python -m venv .venv-ci
                        .venv-ci/bin/pip install --upgrade pip
                        .venv-ci/bin/pip install -r requirements.txt
                        .venv-ci/bin/pip install pip-audit
                        .venv-ci/bin/pip-audit -r requirements.txt \
                            -f json -o dependency-report.json || true
                    '
                '''
                archiveArtifacts artifacts: 'project4/dependency-report.json',
                                 allowEmptyArchive: true
            }
        }

        stage('Django Check') {
            steps {
                sh '''
                    docker run --rm \
                    -v "$WORKSPACE:/workspace" \
                    -w /workspace/project4 \
                    python:3.11-slim \
                    .venv-ci/bin/python manage.py check
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
                echo 'Project4 pipeline stages completed'
            }
        }
    }

    post {
        success {
            echo 'Project4 CI Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
