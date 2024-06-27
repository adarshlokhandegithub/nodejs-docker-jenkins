pipeline {
    agent any
    
    environment {
        registryCredential = 'ACR' // Credential ID for Azure Container Registry
        registry = 'azjenkinsvmcr.azurecr.io'
        imageName = 'nodejs-app'
        containerPort = 3000
        webAppName = 'jenkinsdeployment'
        resourceGroupName = 'jenkins-rg' // Azure Resource Group
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
                    docker.build("${registry}/${imageName}:${env.BUILD_NUMBER}", '-f Dockerfile .')
                    
                    // Login to Azure Container Registry
                    docker.withRegistry('https://${registry}', env.registryCredential) {
                        // Push built Docker image to Azure Container Registry
                        docker.image("${registry}/${imageName}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy to Azure Web App') {
            steps {
                script {
                    // Deploy Docker image to Azure Web App
                    sh "az webapp config container set --name ${webAppName} --resource-group ${resourceGroupName} --docker-custom-image-name ${registry}/${imageName}:${env.BUILD_NUMBER}"

                    // Restart Azure Web App to apply changes
                    sh "az webapp restart --name ${webAppName} --resource-group ${resourceGroupName}"
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
    }
}
