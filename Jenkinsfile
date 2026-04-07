pipeline {
    agent any
    
    environment {
        BRANCH_STRATEGY = "${env.GIT_BRANCH == 'main' ? 'release' : 'feature'}"
    }
    
    stages {
        stage('Strategy Router') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'main') {
                        echo "=== PRODUCTION PIPELINE: полные тесты + деплой ==="
                    } else {
                        echo "=== FEATURE PIPELINE: быстрые тесты, нет деплоя ==="
                    }
                }
            }
        }
        
        stage('Configure Feature Flags') {
            steps {
                script {
                    if (env.GIT_BRANCH =~ /^feature\/.*/) {
                        sh 'echo "FEATURE_NEW_UI=true" > .env'
                    } else {
                        sh 'echo "FEATURE_NEW_UI=false" > .env'
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    // Проверка: есть ли тесты в изменённых файлах
                    def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
                    if (changedFiles.contains('.php') && !changedFiles.contains('Test.php')) {
                        error("❌ Изменения в коде без соответствующих тестов!")
                    }
                    
                    // Проверка: размер Docker-образа не более 500MB
                    def imageSize = sh(script: "docker images myapp:${BUILD_NUMBER} --format '{{.Size}}' | sed 's/MB//'", returnStdout: true).trim()
                    if (imageSize.toInteger() > 500) {
                        error("❌ Размер образа превышает 500MB!")
                    }
                }
            }
        }
        
        // Добавьте остальные этапы (Build, Test, Deploy) здесь
        stage('Build') {
            steps {
                sh 'echo "Building Docker image..."'
                // sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
                // sh 'phpunit tests/'
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'main') {
                        sh 'echo "Deploying to production..."'
                        // Здесь команды деплоя
                    } else {
                        sh 'echo "Skipping deploy for feature branch"'
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh 'echo "Pipeline finished"'
        }
        success {
            sh 'echo "✅ Build successful!"'
        }
        failure {
            sh 'echo "❌ Build failed!"'
        }
    }
}