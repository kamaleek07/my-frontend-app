pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'baymax786/my-frontend'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    url: ''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Update Manifest Repository') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', 
                                                    usernameVariable: 'GITHUB_USERNAME', 
                                                    passwordVariable: 'GITHUB_TOKEN')]) {
                        sh """
                            # Remove existing directory if it exists
                            rm -rf my-frontend-manifests
                            
                            # Clone fresh
                            git clone https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/my-frontend-manifests.git
                            cd my-frontend-manifests
                            
                            # Update the image tag in deployment.yaml
                            sed -i "s|image: baymax786/my-frontend:.*|image: baymax786/my-frontend:${DOCKER_TAG}|g" deployment.yaml
                            
                            git config user.email "lucy2113@gmail.com"
                            git config user.name "Jenkins"
                            
                            git add deployment.yaml
                            git commit -m "Update image to version ${DOCKER_TAG}"
                            git push https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/my-frontend-manifests.git main
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}