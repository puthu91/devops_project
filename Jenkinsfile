pipeline {
    agent any

    // Define environment variables
    environment {
        // Replace with your Docker Hub username and repo name
        DOCKER_IMAGE_NAME = "puthu031991/devops_project"
        // Get Jenkins build number for unique tagging
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        // Credential ID for Docker Hub login (defined in Jenkins UI)
        DOCKERHUB_CREDENTIALS = 'docker-hub-credentials'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image using the Dockerfile in the current directory
                    // and tag it with the specific build number
                    docker.build("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Authenticate and push the image to Docker Hub
                    // The 'withRegistry' block uses the stored credentials
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
                        docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS) {
                            def img = docker.image("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}")
                            img.push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Dynamically replace the placeholder tag in the YAML file with the current build tag
                    // and apply the configuration to the Kubernetes cluster
                    sh "sed -i 's|BUILD_TAG_PLACEHOLDER|${IMAGE_TAG}|' k8s/deployment.yaml"
                    
                    // Apply the updated YAML using kubectl (assumes kubectl is installed and configured on the Jenkins agent)
                    sh "kubectl apply -f k8s/deployment.yaml"

                    // (Optional) Clean up the modified file to ensure repository integrity
                    sh "git checkout k8s/deployment.yaml" 
                }
            }
        }
    }
}
