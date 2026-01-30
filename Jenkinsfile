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
                // Configure Docker pour utiliser le Docker daemon de Minikube
                sh 'eval $(minikube docker-env)'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Taguer l'image avec BUILD_NUMBER pour avoir une version unique
                    env.IMAGE_TAG = "${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "🐳 Build de l'image Docker avec tag ${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Clean Kubernetes') {
            steps {
                echo "🧹 Nettoyage des anciens pods et services"
                sh "kubectl delete deployment ${DEPLOYMENT_NAME} --ignore-not-found"
                sh "kubectl delete service ${SERVICE_NAME} --ignore-not-found"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Déploiement Kubernetes"
                // Applique le nouveau deployment et service
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                // Met à jour le déploiement avec la nouvelle image
                sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${IMAGE_TAG}"
                // Attend que tous les pods soient Running
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
            }
        }

        stage('Check Service') {
            steps {
                echo "✅ Vérification du service"
                script {
                    def MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
                    sh "sleep 5" // attendre que le service soit prêt
                    sh "curl -f http://${MINIKUBE_IP}:${NODE_PORT}"
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline terminé avec succès ! Application déployée sur Kubernetes"
        }
        failure {
            echo "❌ Le pipeline a échoué (tests ou déploiement)"
        }
    }
}
