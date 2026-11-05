
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '839977475583.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = '839977475583.dkr.ecr.ap-south-1.amazonaws.com/devops-flask-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
    steps {
        sh '''
            set -eu

            rm -rf .venv
            python3 -m venv .venv

            .venv/bin/python -m pip install --no-cache-dir --upgrade pip
            .venv/bin/python -m pip install --no-cache-dir -r requirements.txt
            .venv/bin/python -m pip install --no-cache-dir pytest
        '''
    }
}

        stage('Unit Tests') {
            steps {
                sh '.venv/bin/python -m pytest -q'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t devops-flask-app:${BUILD_NUMBER} \
                        -t devops-flask-app:latest .
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu

                    docker rm -f flask-ci-test >/dev/null 2>&1 || true

                    cleanup() {
                        docker rm -f flask-ci-test >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker run -d --name flask-ci-test \
                        -p 127.0.0.1:5001:5000 \
                        devops-flask-app:${BUILD_NUMBER}

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:5001/health; then
                            echo "Smoke test passed!"
                            exit 0
                        fi

                        echo "Waiting for Flask application... attempt $i"
                        sleep 2
                    done

                    echo "Smoke test failed. Container logs:"
                    docker logs flask-ci-test
                    exit 1
                '''
            }
        }

        stage('Push to AWS ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        set +x
                        set -eu

                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        export AWS_EC2_METADATA_DISABLED=true

                        aws ecr get-login-password \
                            --region "$AWS_REGION" | \
                            docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                        docker tag \
                            devops-flask-app:${BUILD_NUMBER} \
                            ${ECR_REPOSITORY}:${BUILD_NUMBER}

                        docker tag \
                            devops-flask-app:${BUILD_NUMBER} \
                            ${ECR_REPOSITORY}:latest

                        docker push ${ECR_REPOSITORY}:${BUILD_NUMBER}
                        docker push ${ECR_REPOSITORY}:latest

                        docker logout "$ECR_REGISTRY"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}

