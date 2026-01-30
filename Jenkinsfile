pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-devops-app-2:latest"
        CONTAINER_NAME = "test-python-app-2"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/giksone/gojo2.0.git'
            }
        }

        stage('Use Minikube Docker') {
            steps {
                sh '''
                eval $(minikube docker-env)
                docker info | grep Name
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                eval $(minikube docker-env)
                docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Test Application (Docker)') {
            steps {
                sh '''
                eval $(minikube docker-env)

                # Nettoyage si un conteneur test existe déjà
                docker rm -f $CONTAINER_NAME || true

                # Lancer le conteneur en test
                docker run -d --name $CONTAINER_NAME -p 5001:5001 $IMAGE_NAME

                # Attendre que l'app démarre
                sleep 3

                # Tester l'endpoint
                curl -f http://127.0.0.1:5001/

                # Arrêter le conteneur de test
                docker stop $CONTAINER_NAME
                docker rm $CONTAINER_NAME
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml

                kubectl set image deployment/python-app-2 python-app-2=$IMAGE_NAME
                kubectl rollout restart deployment/python-app-2
                '''
            }
        }

        stage('Check Status') {
            steps {
                sh '''
                kubectl rollout status deployment/python-app-2
                kubectl get pods
                kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD gojo2.0 terminé avec succès"
        }
        failure {
            echo "❌ Le pipeline a échoué (tests ou déploiement)"
        }
    }
}
