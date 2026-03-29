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
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
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
                            rm -rf my-frontend-manifests
                            
                            # ✅ Correct repo URL + correct path
                            git clone https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/my-frontend-manifests.git
                            
                            cd my-frontend-manifests
                            
                            # Update image tag
                            sed -i "s|image: kamalee07/my-frontend:.*|image: kamalee07/my-frontend:${DOCKER_TAG}|g" deployment.yaml
                            
                            git config user.email "kamaleek07@gmail.com"
                            git config user.name "Jenkins"
                            
                            git add deployment.yaml
                            git commit -m "Update image to version ${DOCKER_TAG}" || echo "No changes to commit"
                            
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