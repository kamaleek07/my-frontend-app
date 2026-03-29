pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'kamalee07/my-frontend'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: 'master', 
                    url: 'https://github.com/kamaleek07/my-frontend-app.git'
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
                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials', 
                        usernameVariable: 'GITHUB_USERNAME', 
                        passwordVariable: 'GITHUB_TOKEN'
                    )]) {
                        
                        sh '''
                            # Remove old folder
                            rm -rf my-frontend-manifests
                            
                            # Clone WITHOUT token (safe)
                            git clone https://github.com/$GITHUB_USERNAME/my-frontend-manifests.git
                            cd my-frontend-manifests
                            
                            # Update image tag
                            sed -i "s|image: kamalee07/my-frontend:.*|image: kamalee07/my-frontend:${DOCKER_TAG}|g" deployment.yaml
                            
                            # Git config
                            git config user.email "kamaleek07@gmail.com"
                            git config user.name "Jenkins"
                            
                            # Commit changes
                            git add deployment.yaml
                            git commit -m "Update image to version ${DOCKER_TAG}" || echo "No changes to commit"
                            
                            # Set authenticated remote (SECURE)
                            git remote set-url origin https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/$GITHUB_USERNAME/my-frontend-manifests.git
                            
                            # Push
                            git push origin master
                        '''
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}