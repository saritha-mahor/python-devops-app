
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        APP_IMAGE = 'python-devops-app:1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Project Files') {
            steps {
                sh '''
                    set -eu
                    test -f app.py
                    test -f requirements.txt
                    test -f Dockerfile
                    test -f tests/test_app.py
                    echo "Project files validated."
                '''
            }
        }

        stage('Python Dependencies') {
            steps {
                sh '''
                    set -eu
                    python3 --version
                    python3 -m venv .venv
                    .venv/bin/python -m pip install --upgrade pip
                    .venv/bin/python -m pip install -r requirements.txt
                    .venv/bin/python -m pip install pytest
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''
                    set -eu
                    .venv/bin/python -m pytest -v \
                        --junitxml=test-results.xml
                '''
            }
            post {
                always {
                    junit(
                        testResults: 'test-results.xml',
                        allowEmptyResults: true
                    )
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -eu
                    docker build -t "$APP_IMAGE" .
                    docker image inspect "$APP_IMAGE"
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    set -eu
                    if command -v trivy >/dev/null 2>&1; then
                        trivy image \
                            --severity HIGH,CRITICAL \
                            --exit-code 0 \
                            "$APP_IMAGE"
                    else
                        echo "Trivy is not installed; scan skipped."
                    fi
                '''
            }
        }

    }

    post {
        success {
            echo 'CI succeeded: tests passed and the image was prepared.'
        }
        failure {
            echo 'CI failed. Review the first failing stage.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
