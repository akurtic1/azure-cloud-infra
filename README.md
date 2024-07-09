# 🚀 Azure-cloud-infra

## 📋 Table of Contents

- [Introduction](#introduction)
- [Technologies Used](#features)
- [Summary](#summary)

## 🌟 Introduction

This project involves setting up a complete DevOps pipeline, managing cloud-based infrastructure,
and ensuring security and automation within a Microsoft Azure environment. The project will be broken down into several tasks.

## ✨ Features

Technologies used:

+ Docker - Dockerized application using Nginx for serving static content
+ GitHub Actions - Automated build and deployment using GitHub Actions
+ Azure App Service - Deployed to Azure App Service from Azure Container Registry
+ Azure Container Registry

## ☁️ Getting started:

To get started with this project, follow these steps:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/akurtic1/azure-cloud-infra
   cd my-dockerized-app

docker build -t myapp:latest ./web-app
docker run -d -p 8080:80 myapp:latest

## ☁️ Building & Continuous Integration

In this step, I used the free template for a simple static web application.
After that, I created a workflow file:

name: CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Run a one-line script
      run: echo Hello, world!

After this, I commited and pushed the changes.

## ☁️ Dockerfile

In this task, I created a dockerfile with the following content:

FROM nginx:alpine
COPY . /usr/share/nginx/html

After that, I built a docker Image and pushed the Docker image to Docker hub.

docker tag myapp:latest mikkasa/myapp:latest
docker push mikkasa/myapp:latest

## ☁️ Cloud Deployment

In this task, I created a resource "Azure Container Registry" and pushed the Docker images
to the Azure Container Registry that I created.

docker tag myapp:latest azurecloudinfra.azurecr.io/myapp:latest
docker push azurecloudinfra.azurecr.io/myapp:latest

After that, I have deployed the Azure App service and I used the images that I pushed
to Azure Container Registry.
You can preview the following URL to confirm that nginx web server is working: https://mywebappinfra.azurewebsites.net/

**I used Command prompt to provide the resources**

## ☁️ Infrastructure as Code 

This is the ARM template that I have developed. This template automates the deployment of an Azure App Service
with the Web App running a specific Docker image. Since this is made for a personal use, I used the Free Tier of the Azure App Service.
```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2023-12-01",
      "name": "azurecloudservice",
      "location": "westeurope",
      "sku": {
        "name": "F1",
        "tier": "Free"
      },
      "properties": {}
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2023-12-01",
      "name": "mywebappinfra",
      "location": "westeurope",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', 'azurecloudservice')]"
      ],
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', 'azurecloudservice')]",
        "siteConfig": {
          "linuxFxVersion": "DOCKER|azurecloudregistry.azurecr.io/myapp:latest"
        }
      }
    }
  ]
}
```
## ☁️ Continuous Deployment

In this task, I achieved the automated deployment of the application to Azure whenever
changes are pushed to the main branch of the GitHub repository. The YAML file is down below:

name: CI/CD

on: [push]
#tete2te33222
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Build Docker image
      run: docker build -t myapp:latest .

    - name: Log in to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Log in to Azure Container Registry
      run: |
        echo ${{ secrets.CONTAINER_REGISTRY_PASSWORD }} | docker login azurecloudregistry.azurecr.io --username ${{ secrets.CONTAINER_REGISTRY_USERNAME }} --password-stdin

    - name: Tag Docker image
      run: docker tag myapp:latest azurecloudregistry.azurecr.io/myapp:latest

    - name: Push Docker image to ACR
      run: docker push azurecloudregistry.azurecr.io/myapp:latest

    - name: Deploy to Azure Web App
      run: az webapp create --resource-group azure-cloud-infra --plan azurecloudservice --name mywebappinfra --deployment-container-image-name azurecloudregistry.azurecr.io/myapp:latest

