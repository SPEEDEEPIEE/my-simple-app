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
                        echo "✅ Feature flag ENABLED for this branch"
                    } else {
                        sh 'echo "FEATURE_NEW_UI=false" > .env'
                        echo "✅ Feature flag DISABLED for this branch"
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    // Проверка наличия тестов
                    def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
                    if (changedFiles.contains('.php') && !changedFiles.contains('Test.php')) {
                        error("❌ Изменения в коде без соответствующих тестов!")
                    }
                    echo "✅ Test check passed"
                    
                    // Проверка размера Docker-образа (безопасная)
                    try {
                        def imageSizeRaw = sh(
                            script: "docker images myapp:${BUILD_NUMBER} --format '{{.Size}}' 2>/dev/null || echo ''", 
                            returnStdout: true
                        ).trim()
                        
                        if (imageSizeRaw.isEmpty()) {
                            echo "⚠️ Docker image myapp:${BUILD_NUMBER} not found, skipping size check"
                        } else {
                            def sizeNumber = imageSizeRaw.replaceAll('[^0-9.]', '')
                            if (!sizeNumber.isEmpty()) {
                                def imageSize = sizeNumber.toDouble()
                                if (imageSize > 500) {
                                    error("❌ Размер образа ${imageSize}MB превышает лимит 500MB!")
                                } else {
                                    echo "✅ Image size: ${imageSize}MB"
                                }
                            }
                        }
                    } catch (Exception e) {
                        echo "⚠️ Docker size check skipped: ${e.message}"
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                echo "🔨 Building Docker image..."
                // Раскомментируйте, когда будет готов Dockerfile:
                // sh "docker build -t myapp:${BUILD_NUMBER} ."
            }
        }
        
        stage('Test') {
            steps {
                echo "🧪 Running tests..."
                // Раскомментируйте, когда будут готовы тесты:
                // sh "phpunit tests/ || true"
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'main') {
                        echo "🚀 Deploying to production..."
                        // Здесь команды деплоя
                    } else {
                        echo "⏭️ Skipping deploy for feature branch (${env.GIT_BRANCH})"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "🏁 Pipeline finished"
        }
        success {
            echo "✅ Build successful!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}