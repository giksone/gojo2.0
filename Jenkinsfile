pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-devops-app-2:latest"
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

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                # Mettre à jour l'image du déploiement existant
                kubectl set image deployment/python-app-2 python-app-2=$IMAGE_NAME

                # Forcer le redéploiement pour récupérer la nouvelle image
                kubectl rollout restart deployment/python-app-2
                '''
            }
        }

        stage('Check Status') {
            steps {
                sh '''
                kubectl get pods
                kubectl get services
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Project 2 terminé avec succès"
        }
        failure {
            echo "❌ Erreur dans le pipeline Project 2"
        }
    }
}
