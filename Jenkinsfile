// Jenkinsfile
pipeline {
    agent any
    
    tools {
        // Убедитесь, что 'NodeJS_LTS' настроен в Global Tool Configuration
        nodejs 'NodeJS_LTS'
    }
    
    environment {
        // Каждое объявление переменной должно быть на отдельной строке
        BRANCH_LOWER = "${env.BRANCH_NAME.toLowerCase()}"
        APP_PORT = '3000'
        CONTAINER_NAME = "${BRANCH_LOWER}-app"
        DOCKER_TAG = "${BRANCH_LOWER}:v1.0"
        DOCKER_REPO = "ekxtz/myrepostudy"
    }
    
    stages {
        stage('Determine Environment & Port') {
            steps {
                script {
                    // Условная логика для установки порта
                    if (env.BRANCH_LOWER == 'dev') {
                        env.APP_PORT = '3001'
                        echo "Running on DEV branch. Setting port to ${env.APP_PORT}."
                    } else if (env.BRANCH_LOWER == 'main') {
                        env.APP_PORT = '3000'
                        echo "Running on MAIN branch. Setting port to ${env.APP_PORT}."
                    }
                }
            }
        }
        
        stage('Checkout & Build') {
            steps {
                checkout scm
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'echo "Running application tests..."'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Tags image as 'nodemain:v1.0' or 'nodedev:v1.0'
                    docker.build("node${BRANCH_LOWER}:v1.0", ".")
                }
            }
        }
        
        stage('Deploy (Local)') {
            steps {
                script {
                    // Остановка и удаление контейнера
                    sh "docker stop ${env.CONTAINER_NAME} || true"
                    sh "docker rm ${env.CONTAINER_NAME} || true"
                    
                    // Развертывание
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 node${BRANCH_LOWER}:v1.0"
                    echo "Application deployed to http://localhost:${env.APP_PORT}"
                }
            }
        }
        
        // ADVANCED TASK: Push to DockerHub
        stage('Push to DockerHub') {
            when { expression { return env.BRANCH_LOWER == 'main' || env.BRANCH_LOWER == 'dev' } }
            steps {
                // Requires 'docker-hub-credentials' ID set up in Jenkins
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: '') {
                    sh "docker tag node${BRANCH_LOWER}:v1.0 ${DOCKER_REPO}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_REPO}:${DOCKER_TAG}"
                }
            }
        }
        
        // ADVANCED TASK: Trigger subsequent deployment pipelines
        // ВНИМАНИЕ: Пайплайны "Deploy_to_main" и "Deploy_to_dev" не существуют,
        // поэтому этот этап вызовет сбой, пока вы их не создадите (Шаг 7).
        stage('Trigger CD Pipeline') {
            when { expression { return env.BRANCH_LOWER == 'main' || env.BRANCH_LOWER == 'dev' } }
            steps {
                // job: "${env.BRANCH_LOWER}__CD_deploy_manual", wait: false
                // Использование имени из задания "CD_deploy_manual"
                build job: "CD_deploy_manual", parameters: [[$class: 'StringParameterValue', name: 'BRANCH_TO_DEPLOY', value: env.BRANCH_LOWER]], wait: false
            }
        }
    }
}
