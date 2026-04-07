pipeline {
    agent any

stage('Quality Gate') {
    steps {
        script {
            // Проверка: есть ли тесты в изменённых файлах
            def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
            if (changedFiles.contains('.php') && !changedFiles.contains('Test.php')) {
                error(' Изменения в коде без соответствующих тестов!')
            }
            
            // Проверка: размер Docker-образа не более 500MB
            def imageSize = sh(script: "docker images ${DOCKER_IMAGE}:${BUILD_NUMBER} --format '{{.Size}}'", returnStdout: true).trim()
            if (imageSize.contains('GB') || imageSize.toInteger() > 500) {
                error(' Размер образа превышает лимит!')
            }
        }
    }
}


    environment {
        DOCKER_IMAGE = '23038/my-simple-app'
        DOCKER_CREDENTIALS_ID = '2303823026'
        BRANCH_STRATEGY = "${env.GIT_BRANCH == 'main' ? 'release' : 'feature'}"
    }

    stage('Configure Feature Flags') {
    steps {
        script {
            if (env.GIT_BRANCH =~ /^feature\/.*/) {
                sh 'echo "FEATURE_NEW_UI=true" >> .env'
            } else {
                sh 'echo "FEATURE_NEW_UI=false" >> .env'
            }
        }
    }
}


    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', 
                url: 'https://github.com/SPEEDEEPIEE/my-simple-app.git'
                script {
                    if (env.GIT_BRANCH == 'master') {
                        // Продакшен-пайплайн: полные тесты + безопасность
                        sh 'echo "Running production pipeline"'
                    } else {
                        // Feature-пайплайн: быстрые тесты для обратной связи
                        sh 'echo "Running feature pipeline"'
                    }
                }   
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Заменяем версию в HTML
                    sh "sed -i 's/BUILD_VERSION/${BUILD_NUMBER}/' index.html"
                    
                    // Собираем образ
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    // Простой тест - проверяем что контейнер запускается
                    sh """
                    docker run -d --name test-container ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    sleep 5
                    docker ps | grep test-container
                    docker stop test-container
                    docker rm test-container
                    """
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    // Останавливаем старый контейнер
                    sh 'docker stop running-app || true'
                    sh 'docker rm running-app || true'
                    
                    // Запускаем новый
                    sh """
                    docker run -d \
                    -p 80:80 \
                    --name running-app \
                    ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        success {
            echo ' CI/CD цепочка успешно завершена!'
            echo "Приложение доступно по адресу: http://10.20.30.31:8081"
        }
        failure {
            echo ' Сборка или деплой завершились ошибкой'
        }
    }
}
