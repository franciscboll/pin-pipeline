pipeline {
    agent any

    environment {
        // Define the Docker image name
        DOCKER_IMAGE_NAME = "franciscoboll/simple-nodejs"
    }

    stages {

        stage('Lint & Security Scan') {
            steps {
                 script {
                     // Install project dependencies to ensure linting can run
                     sh "npm install"

                     // Run linting to check for code style and potential issues
                     sh "npm run lint"

                     // Perform a security scan on the Docker image using Trivy
                     // This checks for vulnerabilities in OS packages and application dependencies
                     sh "docker run --rm aquasec/trivy image ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                 }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Authenticate with Docker Hub using stored credentials
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh "echo ${DOCKER_HUB_PASSWORD} |  docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run the container and execute tests inside it
                    sh " docker run --rm ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} npm test"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Authenticate again with Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"

                    // Push the newly built Docker image
                        sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        
                    // If the build is from the main branch, tag the image as 'latest' and push it
                        if (env.BRANCH_NAME == 'main') {
                            sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
                            sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                try {
                    // Cleanup: Remove the built Docker image from the local machine
                    sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                } catch (Exception e) {
                    echo 'Failed to remove Docker image.'
                }
            }
        }
    }
}
