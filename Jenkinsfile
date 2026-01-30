pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-devops-app-2"
        DEPLOYMENT_NAME = "python-app-2"
        SERVICE_NAME = "python-service-2"
        NODE_PORT = "30001"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checkout du repo Git"
                git url: 'https://github.com/giksone/gojo2.0.git', branch: 'main'
            }
        }

        stage('Setup Minikube Docker Env') {
            steps {
                echo "⚡ Configurer Docker pour Minikube"
                sh 'eval $(minikube docker-env)'
            }
        }

        stage('Clean Old Resources') {
            steps {
                echo "🧹 Suppression des anciens déploiements, services et images"
                sh "kubectl delete deployment ${DEPLOYMENT_NAME} --ignore-not-found"
                sh "kubectl delete service ${SERVICE_NAME} --ignore-not-found"
                sh "kubectl delete pod -l app=${DEPLOYMENT_NAME} --ignore-not-found"
                sh "docker rmi -f ${IMAGE_NAME}:latest || true"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Build de l'image Docker"
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Déploiement Kubernetes"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
            }
        }

        stage('Check Service') {
            steps {
                echo "✅ Vérification du service"
                script {
                    def MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
                    sh "sleep 5"
                    sh "curl -f http://${MINIKUBE_IP}:${NODE_PORT}"
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline terminé avec succès !"
        }
        failure {
            echo "❌ Le pipeline a échoué"
        }
    }
}
