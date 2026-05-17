pipeline {
    agent any
    
    environment {
        // Docker Hub bilgileri
        DOCKER_HUB_REPO = 'cab6y/web-uygulamam'
        DOCKER_CREDENTIALS_ID = 'cab6y'
        
        // Kubernetes
        DEPLOYMENT_NAME = 'web-uygulamam'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('🔍 Checkout') {
            steps {
                echo "📂 GitHub'dan kod çekiliyor..."
                checkout scm
            }
        }
        
        stage('🐳 Docker Build') {
            steps {
                script {
                    echo "🏗️  Docker imajı oluşturuluyor..."
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('🔐 Docker Login & Push') {
            steps {
                script {
                    echo "📤 Docker Hub'a yükleniyor..."
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage('☸️  Kubernetes Deploy') {
            steps {
                script {
                    echo "🚀 Kubernetes deployment güncelleniyor..."
                    
                    // Deployment var mı kontrol et
                    sh """
                        if ! kubectl get deployment ${DEPLOYMENT_NAME} &> /dev/null; then
                            echo "⚠️  Deployment bulunamadı, oluşturuluyor..."
                            kubectl apply -f deployment.yaml
                        fi
                    """
                    
                    // Yeni imajı set et
                    sh """
                        kubectl set image deployment/${DEPLOYMENT_NAME} \
                            ${DEPLOYMENT_NAME}=${DOCKER_HUB_REPO}:${IMAGE_TAG} \
                            --record
                    """
                    
                    // Rollout'u bekle
                    sh """
                        kubectl rollout status deployment/${DEPLOYMENT_NAME} --timeout=180s
                    """
                }
            }
        }
        
        stage('✅ Verify') {
            steps {
                script {
                    echo "🔍 Deployment durumu kontrol ediliyor..."
                    sh """
                        echo "📊 Pod Durumu:"
                        kubectl get pods -l app=${DEPLOYMENT_NAME} -o wide
                        
                        echo ""
                        echo "📜 Rollout Geçmişi:"
                        kubectl rollout history deployment/${DEPLOYMENT_NAME} | tail -5
                        
                        echo ""
                        echo "🎉 Deployment başarılı!"
                        echo "🏷️  Yeni imaj: ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline başarıyla tamamlandı!'
        }
        failure {
            echo '❌ Pipeline başarısız oldu!'
        }
        always {
            // Docker imajlarını temizle (disk dolmasın)
            sh 'docker system prune -f || true'
        }
    }
}