pipeline {
    agent any
    
    environment {
        registryCredential = 'ACR' // Credential ID for Azure Container Registry
        dockerImage = 'azjenkinsvmcr.azurecr.io/nodejs-app' // Name of your Docker image
        appName = 'jenkinsdeployment' // Azure Web App name
        resourceGroup = 'jenkins-rg' // Azure Resource Group
        dockerfilePath = './Dockerfile' // Path to your Dockerfile in the repo
        azureCredentials = credentials('AzureServicePrincipal') // Azure credentials ID configured in Jenkins
    }
    
    stages {
        stage('Prepare') {
            steps {
                // Clean workspace
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/adarshlokhandegithub/nodejs-docker-jenkins']])
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // Build Docker image using Dockerfile
                    def dockerImage = docker.build(env.dockerImage, "--file ${env.dockerfilePath} .")
                    
                    // Login to Azure Container Registry
                    docker.withRegistry('https://azjenkinsvmcr.azurecr.io', env.registryCredential) {
                        // Push built Docker image to Azure Container Registry
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deployment') {
            steps {
                script {
                    // Azure CLI login using Service Principal credentials stored in Jenkins
                    withCredentials([azureServicePrincipal(env.azureCredentials)]) {
                        sh "az login --service-principal -u \${AZURE_CLIENT_ID} -p \${AZURE_CLIENT_SECRET} --tenant \${AZURE_TENANT_ID}"
                        // Set custom Docker image for Azure Web App
                        sh "az webapp config container set --name ${env.appName} --resource-group ${env.resourceGroup} --docker-custom-image-name ${env.dockerImage}:latest"
                    }
                }
            }
        }
    }  
    
    post {
        success {
            echo 'Pipeline successfully completed!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Cleanup steps if necessary
            cleanWs()
        }
    }
}
