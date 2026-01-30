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

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "🐳 Build de l'image Docker avec tag ${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Clean Kubernetes') {
            steps {
                echo "🧹 Suppression des anciens déploiements et services"
                sh "kubectl delete deployment ${DEPLOYMENT_NAME} --ignore-not-found"
                sh "kubectl delete service ${SERVICE_NAME} --ignore-not-found"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Déploiement Kubernetes"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
                sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${IMAGE_TAG}"
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
            echo "❌ Le pipeline a échoué (tests ou déploiement)"
        }
    }
}
