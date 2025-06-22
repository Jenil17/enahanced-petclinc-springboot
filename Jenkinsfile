pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "springboot"
        IMAGE_TAG = "latest"
        ACR_NAME = "mycontainerregistery1712"
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        TENANT_ID = "e36985c9-aaad-40b0-b7e9-d1617ac398f6"
        RESOURCE_GROUP = "Project1"
        CLUSTER_NAME = "demo-project"
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/Jenil17/enahanced-petclinc-springboot.git'
            }
        }

        stage('Maven Package') {
            steps {
                echo 'Building Maven Package'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Azure Login to ACR') {
            steps {
                withCredentials([
                    azureServicePrincipal(
                        credentialsId: '3eab5c61-ad91-4567-9d1c-40535f6f8048',
                        subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                        clientIdVariable: 'AZURE_CLIENT_ID',
                        clientSecretVariable: 'AZURE_CLIENT_SECRET',
                        tenantIdVariable: 'AZURE_TENANT_ID'
                    )
                ]) {
                    sh '''
                    echo "Logging into Azure and ACR"
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                    az acr login --name $ACR_NAME
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker Image to ACR'
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
                docker push ${FULL_IMAGE_NAME}
                '''
            }
        }

        stage('AKS Login') {
            steps {
                withCredentials([
                    azureServicePrincipal(
                        credentialsId: '3eab5c61-ad91-4567-9d1c-40535f6f8048',
                        subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                        clientIdVariable: 'AZURE_CLIENT_ID',
                        clientSecretVariable: 'AZURE_CLIENT_SECRET',
                        tenantIdVariable: 'AZURE_TENANT_ID'
                    )
                ]) {
                    sh '''
                    echo "Connecting to AKS Cluster"
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
                    '''
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                echo 'Deploying to AKS'
                sh '''
                kubectl apply -f k8s/sprinboot-deployment.yaml
                echo 'Deployment completed successfully!'
                '''
            }
        }
    }
}
