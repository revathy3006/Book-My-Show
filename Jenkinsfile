pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPOSITORY = '129950599327.dkr.ecr.us-east-1.amazonaws.com/bms'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/revathy3006/Book-My-Show.git'

                sh '''
                    echo "Repository contents:"
                    ls -la
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        npm install
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        export NODE_OPTIONS=--openssl-legacy-provider
                        export CI=false
                        npm run build
                    '''
                }
            }
        }

        stage('Jest Test') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                        CI=true npm test -- --watchAll=false --coverage
                    '''
                }
            }
        }

        stage('Generate HTML Test Report') {
            steps {
                archiveArtifacts artifacts: 'bookmyshow-app/coverage/lcov-report/**',
                                 allowEmptyArchive: false
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                    trivy fs --exit-code 0 .
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${ECR_REPOSITORY}:${IMAGE_TAG} \
                        -f bookmyshow-app/Dockerfile \
                        bookmyshow-app
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image \
                        --exit-code 0 \
                        ${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    echo "Logging in to AWS ECR..."

                    aws ecr get-login-password \
                        --region ${AWS_REGION} |
                    docker login \
                        --username AWS \
                        --password-stdin ${ECR_REPOSITORY}

                    echo "Pushing image to ECR..."

                    docker push ${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }
        stage('Deploy to EKS using Ansible') {
            steps {
                sh '''
                    BUILD_NUMBER=${BUILD_NUMBER} \
                    ansible-playbook \
                    -i ansible/inventory \
                    ansible/deploy.yml
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline completed for build ${BUILD_NUMBER}"
        }

        success {
            echo "SUCCESS!"
            echo "Image pushed:"
            echo "${ECR_REPOSITORY}:${IMAGE_TAG}"
            echo "Application deployed successfully to EKS using Ansible."
        }

        failure {
            echo "Pipeline FAILED. Check the failed stage."
        }
    }
}
