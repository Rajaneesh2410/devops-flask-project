pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '.venv/bin/python -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-flask-app:latest .'
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu

                    docker rm -f flask-ci-test || true

                    cleanup() {
                        docker rm -f flask-ci-test >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker run -d \
                        --name flask-ci-test \
                        -p 127.0.0.1:5001:5000 \
                        devops-flask-app:latest

                    curl --fail --retry 10 \
                        --retry-connrefused --retry-delay 1 \
                        http://localhost:5001/health
                '''
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
