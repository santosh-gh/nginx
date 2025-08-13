minikube service nginx-service --url



# Part 5: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline

    Part1: Manual Deployment using comand line tools (AzCLI, Docker Desktop and kubectl)  
    GitHub: https://github.com/santosh-gh/k8s-01
    YouTube: https://youtu.be/zoJ7MMPVqFY

    Part2: Automated Deployment using Azure DevOps Pipeline
    GitHub: https://github.com/santosh-gh/k8s-02
    YouTube: https://youtu.be/nnomaZVHg9I

    Part3: Automated Infra Deployment using Bicep and Azure DevOps Pipeline
    GitHub: https://github.com/santosh-gh/k8s-03
    YouTube: https://www.youtube.com/watch?v=5PAdDPHn8F8

    Part4: Deploying microservice applications in AKS using Helm Chat
    GitHub: https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

    Part5: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
    GitHub: https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=MnWe2KGRrxg&t=883s

    Part6: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
           Dynamically update the image tag in values.yaml
    GitHub: https://github.com/santosh-gh/k8s-06
    YouTube: https://www.youtube.com/watch?v=Nx0defm8T6g&t=11s

    Part7: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
           Store the helm chart in ACR
           Dynamically update the image tag in values.yaml
           Dynamically update the Chart version in Chart.yaml

    GitHub: https://github.com/santosh-gh/k8s-07
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

# Architesture

![Store Architesture](aks-store-architecture.png)

    # Store front: Web application for customers to view products and place orders.
    # Product service: Shows product information.
    # Order service: Places orders.
    # RabbitMQ: Message queue for an order queue.


# Directory Structure

![Directory Structure](image.png)

# Tetechnology Stack

    Azure Pipelines
    Infra (AzCLI/Bicep)
    AKS
    ACR
    HelmChart
    Helmify

# Advantage of storing Helm Chart in ACR

    Private, encrypted storage.

    Access restricted via Azure AD and RBAC.

    Version Management: Charts can be tagged and versioned.

    Easy rollback to previous versions if needed.

    Security & Compliance    

    Supports Azure Policies, private endpoints, and vulnerability scanning.

    improve performance

# Steps

    1. Infra deployment using AzCLI/Bicep command line or 
       Pipelines azcli-infra-pipeline.yml/bicep-infra-pipeline.yml

    2. Build and push images to ACR: CI Pipelines
       order-pipeline.yml, product-pipeline.yml, store-front-pipeline.yml

    3. Helm install and Helmfy
       https://helm.sh/docs/intro/install/

       https://github.com/arttor/helmify/releases

       Advantages of helm over kubectl

       Helm uses templates with variables, so no need to duplicate YAML files for each environment

       Helm supports versioned releases and can be roll back to a previous release easily

       helm list
       helm rollback online-store 1

       Parameterization per Environment using enverionment  specific values.yaml
       helm install online-store ./helmchart -f dev-values-.yaml
       helm install online-store ./helmchart -f test-values.yaml

       Helm keeps track of installed releases, values, and history
       helm list
       helm get all online-store

    4. App deployment: CD Pipelines
       app-deploy-pipeline.yml

    5. Validate and Access the application

    6. Clean the Azure resources
    
# Infra deployment

    # Login to Azure

        az login
        az account set --subscription=<subscriptionId>
        az account show

    # Show existing resources

        az resource list

    # Create RG, ACR and AKS

        # AzCLI
        ./infra/azcli/script.sh

        OR

        # Bicep
        az deployment sub create --location uksouth --template-file ./infra/bicep/main.bicep --parameters ./infra/bicep/main.bicepparam

    # Connect to cluster

        RESOURCE_GROUP="rg-onlinestore-dev-uksouth-001"
        AKS_NAME="aks-onlinestore-dev-uksouth-001"
        az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME --overwrite-existing

        alias k=kubectl

    # Short name for kubectl

    # Show all existing objects

        k get all   

Docker Build and Push
# Log in to ACR

    ACR_NAME="acronlinestoredevuksouth001"
    az acr login --name $ACR_NAME

# Build and push the Docker images to ACR

    # Order Service
    docker build -t order ./app/order-service 
    docker tag order:latest $ACR_NAME.azurecr.io/order:v1
    docker push $ACR_NAME.azurecr.io/order:v1

    # Product Service
    docker build -t product ./app/product-service 
    docker tag product:latest $ACR_NAME.azurecr.io/product:v1
    docker push $ACR_NAME.azurecr.io/product:v1

    # Store Front Service
    docker build -t store-front ./app/store-front 
    docker tag store-front:latest $ACR_NAME.azurecr.io/store-front:v1
    docker push $ACR_NAME.azurecr.io/store-front:v1

    docker images

# Helm and Helmify

    # helmify 

    helmify -f ./manifests helmchart

    # Helm Deploy

    helm install online-store ./helmchart

    helm uninstall online-store

    # Delete Services using helm        
     
    helm uninstall online-store

# Helm Push and Install    

    OCI refers to the Open Container Initiative, a lightweight, open governance 
    structure that defines open industry standards for container formats and runtimes. 

    export HELM_EXPERIMENTAL_OCI=1
    On Helm v3.8.0 and later, OCI is supported by default — no environment variable needed.

    helm registry login $ACR_NAME.azurecr.io

    helm package ./storehelmchart/config  --version 0.1.0
    helm package ./storehelmchart/rabbitmq  --version 0.1.0
    helm package ./storehelmchart/order  --version 0.1.0
    helm package ./storehelmchart/product  --version 0.1.0
    helm package ./storehelmchart/store-front  --version 0.1.0

    helm push config-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm
    helm push rabbitmq-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm
    helm push order-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm
    helm push product-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm
    helm push store-front-0.1.0.tgz oci://$ACR_NAME.azurecr.io/helm

    helm pull oci://$ACR_NAME.azurecr.io/helm/config --version 0.1.0
    helm pull oci://$ACR_NAME.azurecr.io/helm/rabbitmq --version 0.1.0
    helm pull oci://$ACR_NAME.azurecr.io/helm/order --version 0.1.0
    helm pull oci://$ACR_NAME.azurecr.io/helm/product --version 0.1.0
    helm pull oci://$ACR_NAME.azurecr.io/helm/store-front --version 0.1.0

    helm install config-release oci://$ACR_NAME.azurecr.io/helm/config --version 0.1.0
    helm install rabbitmq-release oci://$ACR_NAME.azurecr.io/helm/rabbitmq --version 0.1.0
    helm install order-release oci://$ACR_NAME.azurecr.io/helm/order --version 0.1.0
    helm install product-release oci://$ACR_NAME.azurecr.io/helm/product --version 0.1.0
    helm install store-front-release oci://$ACR_NAME.azurecr.io/helm/store-front --version 0.1.0

# Clean the k8s namespace

    k delete all --all -n default

# Verify the Deployment

    k get pods
    k get services
    curl <LoadBalancer public IP>:80
    Browse the app using http://<LoadBalancer public IP>:80

# Clean the Azure resources

    az group delete --name rg-onlinestore-dev-uksouth-001 --yes --no-wait