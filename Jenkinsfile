pipeline {
    agent any
    
    tools {
        // Використовуйте інструмент NodeJS, налаштований у Jenkins Global Tools
        nodejs 'NodeJS_LTS'
    }
    
    environment {
        // Ваш Docker Hub репозиторій
        DOCKER_REPO = "ekxtz/myrepostudy" 
        // Примітка: змінні APP_PORT, LOCAL_BUILD_TAG, FULL_REPO_TAG та CONTAINER_NAME
        // створюються динамічно на етапі 'Setup Variables'.
    }
    
    stages {
        
        // 1. Налаштування змінних залежно від імені гілки
        stage('Setup Variables') {
            steps {
                script {
                    env.BRANCH_LOWER = env.BRANCH_NAME.toLowerCase()
                    
                    if (env.BRANCH_LOWER == 'main') {
                        env.APP_PORT = '3000'
                        // Локальний тег для збірки
                        env.LOCAL_BUILD_TAG = 'app-local:main'
                        // Повний тег для відправки в Docker Hub
                        env.FULL_REPO_TAG = "${env.DOCKER_REPO}/nodemain:v1.0" 
                    } else {
                        // Для 'dev' та інших гілок
                        env.APP_PORT = '3001'
                        env.LOCAL_BUILD_TAG = 'app-local:dev'
                        env.FULL_REPO_TAG = "${env.DOCKER_REPO}/nodedev:v1.0"
                    }
                    env.CONTAINER_NAME = "app-${env.BRANCH_LOWER}"
                    echo "Configured for Branch: ${env.BRANCH_NAME} | Port: ${env.APP_PORT}"
                }
            }
        }
        
        // 2. Встановлення залежностей (npm)
        stage('Install Dependencies') {
            steps {
                // Перехід у каталог 'app', де розташовано package.json
                dir('app') {
                    sh 'npm install'
                }
            }
        }
        
        // 3. Запуск тестів
        stage('Test') {
            steps {
                dir('app') {
                    // Припускається, що скрипт 'test' існує у package.json
                    sh 'npm test'
                }
            }
        }
        
        // 4. Збірка Docker-образу
        stage('Docker Build') {
            steps {
                // Перехід у папку 'app', де розташовано Dockerfile
                dir('app') {
                    // Використання --no-cache для чистої збірки
                    sh "docker build --no-cache -t ${env.LOCAL_BUILD_TAG} ."
                }
            }
        }
        
        // 5. Відправлення образу до Docker Hub
        stage('Push to Hub') {
            steps {
                script {
                    // Повторити до 3 разів у разі мережевої помилки
                    retry(3) { 
                        // Використовуємо Jenkins Credentials ID
                        withDockerRegistry(credentialsId: 'docker-hub-credentials', url: '') {
                            
                            // 1. Тегування локального образу повним репозиторним іменем
                            sh "docker tag ${env.LOCAL_BUILD_TAG} ${env.FULL_REPO_TAG}"
                            
                            // 2. Відправлення тегу конкретної версії
                            sh "docker push ${env.FULL_REPO_TAG}"
                            
                            // 3. Тільки для гілки 'main': також відправка 'latest'
                            if (env.BRANCH_LOWER == 'main') {
                                sh "docker tag ${env.LOCAL_BUILD_TAG} ${env.DOCKER_REPO}:latest"
                                sh "docker push ${env.DOCKER_REPO}:latest"
                            }
                        }
                    }
                }
            }
        }
        
        // 6. Локальне розгортання для верифікації
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying container ${env.CONTAINER_NAME} on port ${env.APP_PORT}..."
                    
                    // Зупинка та видалення старого контейнера, якщо він існує
                    sh "docker stop ${env.CONTAINER_NAME} || true"
                    sh "docker rm ${env.CONTAINER_NAME} || true"
                    
                    // Запуск нового контейнера, мапінг порту
                    // ${env.APP_PORT} (зовнішній) -> 3000 (внутрішній)
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 ${env.FULL_REPO_TAG}"
                }
            }
        }
        
        // 7. Автоматичний Trigger (викликає зовнішній job)
        stage('Trigger Auto Deploy Pipeline') {
            // Запускається тільки після успішної збірки у гілці 'main'
            when { 
                expression { return env.BRANCH_LOWER == 'main' } 
            }
            steps {
                echo "Triggering auto deployment for main job: Deploy_to_main"
                // Запуск окремого job'а 'Deploy_to_main' без очікування його завершення
                build job: 'Deploy_to_main', wait: false 
            }
        }
    } // Закриває блок stages {}
} // Закриває блок pipeline {}
