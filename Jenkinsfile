pipeline {
    agent any

    environment {
        AWS_REGION = 'il-central-1'
        ECR_REGISTRY = '039438368691.dkr.ecr.il-central-1.amazonaws.com'
        BACKEND_IMAGE = 'task-manager-backend'
        FRONTEND_IMAGE = 'task-manager-frontend'
        K8S_CONTEXT = 'minikube'
    }

    stages {
        stage('Prepare Version') {
            steps {
                script {
                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                }
                echo "Building version: ${env.IMAGE_TAG}"
            }
        }

        stage('Build Docker Images') {
            steps {
                bat 'docker build -t %BACKEND_IMAGE%:%IMAGE_TAG% backend'
                bat 'docker build -t %FRONTEND_IMAGE%:%IMAGE_TAG% frontend'
            }
        }

        stage('Login to ECR') {
            steps {
                bat '''
                aws ecr get-login-password --region %AWS_REGION% > ecr-login.txt
                set /p ECR_PASSWORD=<ecr-login.txt
                docker login --username AWS --password %ECR_PASSWORD% %ECR_REGISTRY%
                del ecr-login.txt
                '''
            }
        }

        stage('Tag Docker Images') {
            steps {
                bat 'docker tag %BACKEND_IMAGE%:%IMAGE_TAG% %ECR_REGISTRY%/%BACKEND_IMAGE%:%IMAGE_TAG%'
                bat 'docker tag %FRONTEND_IMAGE%:%IMAGE_TAG% %ECR_REGISTRY%/%FRONTEND_IMAGE%:%IMAGE_TAG%'
            }
        }

        stage('Push Images to ECR') {
            steps {
                bat 'docker push %ECR_REGISTRY%/%BACKEND_IMAGE%:%IMAGE_TAG%'
                bat 'docker push %ECR_REGISTRY%/%FRONTEND_IMAGE%:%IMAGE_TAG%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl config use-context %K8S_CONTEXT%

                aws ecr get-login-password --region %AWS_REGION% > ecr-password.txt
                set /p ECR_PASSWORD=<ecr-password.txt

                kubectl delete secret ecr-secret --ignore-not-found
                kubectl create secret docker-registry ecr-secret --docker-server=%ECR_REGISTRY% --docker-username=AWS --docker-password=%ECR_PASSWORD%

                del ecr-password.txt

                kubectl apply -f k8s

                kubectl set image deployment/backend backend=%ECR_REGISTRY%/%BACKEND_IMAGE%:%IMAGE_TAG%
                kubectl set image deployment/frontend frontend=%ECR_REGISTRY%/%FRONTEND_IMAGE%:%IMAGE_TAG%

                kubectl rollout status deployment/backend --timeout=180s
                kubectl rollout status deployment/frontend --timeout=180s
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
