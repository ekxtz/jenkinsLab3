pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS_LTS'
    }
    
    environment {
        DOCKER_REPO = "ekxtz/myrepostudy"
    }
    
    stages {
        stage('Determine Environment & Port') {
            steps {
                script {
                    env.BRANCH_LOWER = env.BRANCH_NAME.toLowerCase()

                    if (env.BRANCH_LOWER == 'dev') {
                        env.APP_PORT = '3001'
                        env.IMAGE_TAG = 'nodedev:v1.0'
                    } else if (env.BRANCH_LOWER == 'main') {
                        env.APP_PORT = '3000'
                        env.IMAGE_TAG = 'nodemain:v1.0'
                    } else {
                        env.APP_PORT = '3002'
                        env.IMAGE_TAG = "node-${env.BRANCH_LOWER}:latest"
                    }
                    
                    env.CONTAINER_NAME = "app-${env.BRANCH_LOWER}"
                    
                    echo "Running on ${env.BRANCH_NAME} branch. Port: ${env.APP_PORT}. Container: ${env.CONTAINER_NAME}"
                }
            }
        }
        
        stage('Checkout & Build') {
            steps {
                dir('app') { 
                    sh 'npm install'
                }
            }
        }
        
        stage('Test') {
            steps {
                dir('app') {
                    sh 'npm test'
                    echo "Running application tests..."
                }
            }
        }

        stage('Cleanup Old Docker Artifacts') {
            steps {
                sh "docker system prune -af"
                echo "Docker cleanup complete."
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.IMAGE_TAG}", ".")
                    echo "Docker image ${env.IMAGE_TAG} built successfully."
                }
            }
        }
        
        stage('Push to DockerHub') {
            when { expression { return env.BRANCH_LOWER == 'main' || env.BRANCH_LOWER == 'dev' } }
            steps {
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: '') {
                    sh "docker tag ${env.IMAGE_TAG} ${env.DOCKER_REPO}:${env.GIT_COMMIT}"
                    sh "docker push ${env.DOCKER_REPO}:${env.GIT_COMMIT}"

                    sh "docker tag ${env.IMAGE_TAG} ${env.DOCKER_REPO}:latest"
                    sh "docker push ${env.DOCKER_REPO}:latest"
                    
                    echo "Docker images pushed to Docker Hub."
                }
            }
        }
        
        stage('Deploy (Local)') {
            steps {
                script {
                    echo "Attempting to deploy ${env.IMAGE_TAG} to port ${env.APP_PORT}..."
                    
                    sh "docker stop ${env.CONTAINER_NAME} || true"
                    sh "docker rm ${env.CONTAINER_NAME} || true"
                    
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 ${env.IMAGE_TAG}"
                    echo "Application deployed to http://localhost:${env.APP_PORT}"
                }
            }
        }
        
        stage('Trigger Auto Deploy Pipeline') {
            when { expression { return env.BRANCH_LOWER == 'main' } }
            steps {
                echo "Triggering auto deployment for main job: Deploy_to_main"
                build job: 'Deploy_to_main', wait: false 
            }
        }
    }
}
