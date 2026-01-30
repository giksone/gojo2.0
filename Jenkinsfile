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

        stage('Test Local Container') {
            steps {
                echo "⚡ Test local du container"
                // Supprime un ancien container si existant
                sh "docker rm -f test-python-app-2 || true"
                // Lance le container pour test
                sh "docker run -d --name test-python-app-2 -p 5001:5001 ${IMAGE_TAG}"
                // Attendre que le container soit prêt
                sh "sleep 5"
                // Vérifier que l'application répond
                sh "curl -f http://127.0.0.1:5001"
                // Stoppe et supprime le container de test
                sh "docker rm -f test-python-app-2"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Déploiement Kubernetes"
                // Supprime ancien déploiement et service si existants
                sh "kubectl delete deployment ${DEPLOYMENT_NAME} --ignore-not-found"
                sh "kubectl delete service ${SERVICE_NAME} --ignore-not-found"

                // Applique le nouveau deployment et service
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"

                // Met à jour le déploiement avec la nouvelle image
                sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${IMAGE_TAG}"
                // Attend que tous les pods soient Running
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
            }
        }

        stage('Check Status') {
            steps {
                echo "✅ Vérification du service"
                script {
                    def MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
                    // Attendre que les pods soient prêts
                    sh "sleep 10"
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
