pipeline {
    agent any
    
    tools {
        // Use the NodeJS tool configured in Jenkins Global Tools
        nodejs 'NodeJS_LTS'
    }
    
    environment {
        // Your Docker Hub repository
        DOCKER_REPO = "ekxtz/myrepostudy" 
    }
    
    stages {
        // 1. Setup variables based on Branch Name
        stage('Setup Variables') {
            steps {
                script {
                    env.BRANCH_LOWER = env.BRANCH_NAME.toLowerCase()
                    
                    if (env.BRANCH_LOWER == 'main') {
                        env.APP_PORT = '3000'
                        // Local tag for building
                        env.LOCAL_BUILD_TAG = 'app-local:main'
                        // Full tag for pushing to Docker Hub
                        env.FULL_REPO_TAG = "${env.DOCKER_REPO}/nodemain:v1.0" 
                    } else {
                        // For 'dev' and others
                        env.APP_PORT = '3001'
                        env.LOCAL_BUILD_TAG = 'app-local:dev'
                        env.FULL_REPO_TAG = "${env.DOCKER_REPO}/nodedev:v1.0"
                    }
                    env.CONTAINER_NAME = "app-${env.BRANCH_LOWER}"
                    echo "Configured for Branch: ${env.BRANCH_NAME} | Port: ${env.APP_PORT}"
                }
            }
        }
        
        // 2. Install Dependencies (npm)
        stage('Install Dependencies') {
            steps {
                // Change directory to 'app' where package.json is located
                dir('app') {
                    sh 'npm install'
                }
            }
        }
        
        // 3. Run Tests
        stage('Test') {
            steps {
                dir('app') {
                    // Assumes 'test' script exists in package.json
                    sh 'npm test'
                }
            }
        }
        
        // 4. Build Docker Image
        stage('Docker Build') {
            steps {
                // Change to 'app' folder where Dockerfile is located
                dir('app') {
                    // Use --no-cache to ensure a clean build and avoid layer corruption issues
                    sh "docker build --no-cache -t ${env.LOCAL_BUILD_TAG} ."
                }
            }
        }
        
        // 5. Push Image to Docker Hub
        stage('Push to Hub') {
            steps {
                script {
                    // Retry up to 3 times in case of network timeout
                    retry(3) { 
                        // Use the credentials ID configured in Jenkins
                        withDockerRegistry(credentialsId: 'docker-hub-credentials', url: '') {
                            
                            // 1. Tag the local image with the full repo name
                            sh "docker tag ${env.LOCAL_BUILD_TAG} ${env.FULL_REPO_TAG}"
                            
                            // 2. Push the specific version tag
                            sh "docker push ${env.FULL_REPO_TAG}"
                            
                            // 3. For 'main' branch only: also push 'latest'
                            if (env.BRANCH_LOWER == 'main') {
                                sh "docker tag ${env.LOCAL_BUILD_TAG} ${env.DOCKER_REPO}:latest"
                                sh "docker push ${env.DOCKER_REPO}:latest"
                            }
                        }
                    }
                }
            }
        }
        
        // 6. Local Deployment
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying container ${env.CONTAINER_NAME} on port ${env.APP_PORT}..."
                    
                    // Stop and remove old container if it exists
                    sh "docker stop ${env.CONTAINER_NAME} || true"
            sh "docker rm ${env.CONTAINER_NAME} || true"
                    
                    // Run the new container
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 ${env.FULL_REPO_TAG}"
                }
            }
        }
    }
}
