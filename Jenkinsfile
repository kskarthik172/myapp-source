pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'jdk-11'
    }

    parameters {
        choice(
            name: 'BRANCH',
            choices: ['main', 'dev', 'release'],
            description: 'Select Git branch'
        )
    }

    environment {
        // Sonar Configuration
        SONAR_SERVER = 'sonarqube-server'
        SONAR_PROJECT_KEY = 'myapp'

        // AWS ECR Configuration
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '420838436623'
        ECR_REPO_NAME = 'hello-java'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${params.BRANCH}"
        FULL_IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/kskarthik172/myapp-source.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    withCredentials([string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        sh """
                            mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Docker Image (Multi-Stage)') {
            steps {
                sh """
                    docker build --no-cache -t ${FULL_IMAGE_NAME} .
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    echo "Logging into AWS ECR..."
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    echo "Pushing image to ECR..."
                    docker push ${FULL_IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
            echo "Image pushed: ${FULL_IMAGE_NAME}"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
