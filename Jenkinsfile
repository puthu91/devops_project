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
        KUBECONFIG_CRED_ID = "kubeconfig-embedded"
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
                // Use withCredentials to access the kubeconfig file securely
                withCredentials([file(credentialsId: KUBECONFIG_CRED_ID, variable: 'KUBECONFIG_PATH')]) {
                    sh '''
                    # Set the KUBECONFIG environment variable
                    export KUBECONFIG="/usr/local/bin/kubectl"
                    # Replace the placeholder in the YAML file with the actual image tag
                    sed -i "s|\\${IMAGE_TAG}|${BUILD_NUMBER}|" train-schedule-kube.yml
                    # Apply the deployment to the Kubernetes cluster using kubectl
                    kubectl apply -f train-schedule-kube.yml
                    # Apply service file if needed
                    # kubectl apply -f train-schedule-kube.yml
                    '''
                }
            }
        }
    }
}
       
