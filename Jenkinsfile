// Jenkinsfile
pipeline { agent any tools {
        // Must match the name configured in Global Tool Configuration (Step 
        // 5 of Phase 3)
        nodejs 'NodeJS_LTS'
    }
    // Define environment variables based on branch
    environment { BRANCH_LOWER = "${env.BRANCH_NAME.toLowerCase()}" APP_PORT 
        = '3000' // Default/Main port CONTAINER_NAME = "${BRANCH_LOWER}-app"
        // Use specific naming pattern for Advanced Task
        DOCKER_TAG = "${BRANCH_LOWER}:v1.0" DOCKER_REPO = 
        "YOUR_DOCKERHUB_USERNAME/app-repo" // <--- **UPDATE THIS**
    }
    stages { stage('Determine Environment & Port') { steps { script { if 
                    (env.BRANCH_LOWER == 'dev') {
                        env.APP_PORT = '3001' echo "Running on DEV branch. 
                        Setting port to ${env.APP_PORT}."
                    } else if (env.BRANCH_LOWER == 'main') {
                        env.APP_PORT = '3000' echo "Running on MAIN branch. 
                        Setting port to ${env.APP_PORT}."
                    }
                }
            }
        }
        stage('Checkout & Build') { steps { checkout scm sh 'npm install'
            }
        }
        stage('Test') { steps { sh 'echo "Running application tests..."'
            }
        }
        stage('Build Docker Image') { steps { script {
                    // Tags image as 'nodemain:v1.0' or 'nodedev:v1.0'
                    docker.build("node${BRANCH_LOWER}:v1.0", ".")
                }
            }
        }
        stage('Deploy (Local)') { steps { script {
                    // Targetted container deletion based on env/branch name
                    sh "docker stop ${env.CONTAINER_NAME} || true" sh 
                    "docker rm ${env.CONTAINER_NAME} || true"
                    // Deploy using the conditional port
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p 
                    ${env.APP_PORT}:3000 node${BRANCH_LOWER}:v1.0" echo 
                    "Application deployed to 
                    http://localhost:${env.APP_PORT}"
                }
            }
        }
        // ADVANCED TASK: Push to DockerHub
        stage('Push to DockerHub') { when { expression { return 
            env.BRANCH_LOWER == 'main' || env.BRANCH_LOWER == 'dev' } } 
            steps {
                // Requires 'docker-hub-credentials' ID set up in Jenkins
                withDockerRegistry(credentialsId: 'docker-hub-credentials', 
                url: '') {
                    sh "docker tag node${BRANCH_LOWER}:v1.0 
                    ${DOCKER_REPO}:${DOCKER_TAG}" sh "docker push 
                    ${DOCKER_REPO}:${DOCKER_TAG}"
                }
            }
        }
        // ADVANCED TASK: Trigger subsequent deployment pipelines
        stage('Trigger CD Pipeline') { when { expression { return 
            env.BRANCH_LOWER == 'main' || env.BRANCH_LOWER == 'dev' } } 
            steps {
                // Requires "Deploy_to_main" and "Deploy_to_dev" jobs to be 
                // created
                build job: "Deploy_to_${env.BRANCH_LOWER}", wait: false
            }
        }
    }
}
