pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-devops-app-2:latest"
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
                echo "🐳 Build de l'image Docker"
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Test Local Container') {
            steps {
                echo "⚡ Test local du container"
                // Supprime un ancien container si existant
                sh "docker rm -f test-python-app-2 || true"
                // Lance le container
                sh "docker run -d --name test-python-app-2 -p 5001:5001 ${IMAGE_NAME}"
                // Test de la réponse
                sh "sleep 5" // attendre que le container soit prêt
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

                // Force le redéploiement (utile si image a le même tag)
                sh "kubectl rollout restart deployment ${DEPLOYMENT_NAME}"
            }
        }

        stage('Check Status') {
            steps {
                echo "✅ Vérification du service"
                script {
                    def MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
                    sh "sleep 10" // attendre que les pods soient Running
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
