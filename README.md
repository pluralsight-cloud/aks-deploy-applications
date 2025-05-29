# Deploy Applications to Azure Kubernetes Service (AKS)

> **Note**: Throughout this course I use the `k` alias as a shorthand for `kubectl`, you can set the `k` alias using the following command:

```bash
alias k=kubectl
```

> **Note**: The samples provided in this repository need to be modified/updated to use your values.

## Table of Contents

1. [Module 1: Preparing for Application Deployment](#module-1-preparing-for-application-deployment)

    1. [Application Deployment Concepts](#application-deployment-concepts)
    2. [Demo: Attach an Azure Container Registry](#demo-attach-an-azure-container-registry)
    3. [Demo: Build and Push Container Images](#build-and-push-container-images)

2. [Module 2: Deploying Applications](#module-2-deploying-applications)

    1. [Customizing Applications for Deployment](#customizing-applications-for-deployment)
    2. [Demo: Deploy Applications Manually](#demo-deploy-applications-manually)
    3. [Demo: Applying Kustomizations](#demo-applying-kustomizations)
    4. [Demo: Deploying Applications with Helm](#demo-deploying-applications-with-helm)
    5. [Automated Application Deployment Methods](#automated-application-deployment-methods)
    6. [Demo: Deploying Applications with Flux](#demo-deploying-applications-with-flux)
    7. [Demo: Deploying Applications with Argo CD](#demo-deploying-applications-with-argo-cd)

3. [Module 3: Integrating with Azure Services](#module-3-integrating-with-azure-services)

    1. [Kubernetes Storage Concepts](#kubernetes-storage-concepts)
    2. [Demo: Use Azure Files with AKS](#demo-use-azure-files-with-aks)
    3. [AKS Identity Concepts](#aks-identity-concepts)
    4. [Demo: Configure Workload Identity Federation with AKS](#demo-configure-workload-identity-federation-with-aks)
    5. [Demo: Use Azure Key Vault with AKS](#demo-use-azure-key-vault-with-aks)

## Module 1: Preparing for Application Deployment

### Application Deployment Concepts

- No additional resources.

### Demo: Attach an Azure Container Registry

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-create-an-azure-container-registry --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/1.2/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Authenticate with Azure Container Registry (ACR) from Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?tabs=azure-cli#working-with-acr--aks)
- In the Cloud Playground Sandbox, you can use [Image Pull Secret](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-kubernetes) as an alternative to using an attached Container Registry.
- [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

#### Code Snippets

- (Sample) Set variables used for the sample commands:

    ```bash
    RG=rg-create-an-azure-container-registry
    ACR=cr$RANDOM
    SKU=basic
    ```

- (Sample) Create an Azure Container Registry:

    ```bash
    az acr create \
    --name $ACR \
    --resource-group $RG \
    --sku $SKU
    ```

- (Sample) Create an AKS cluster with an attached Container Registry:

    ```bash
    az aks create \
    --name NewCluster \
    --resource-group $RG \
    --attach-acr $ACR \
    --generate-ssh-keys
    ```

- (Sample) Attach a Container Registry to an existing AKS Cluster:

    ```bash
    az aks update \
    --name ExistingCluster \
    --resource-group $RG \
    --attach-acr $ACR
    ```

- (Sample) Validate an attached Container Registry:

    ```bash
    az aks check-acr \
    --name ExistingCluster \
    --resource-group $RG \
    --acr $ACR
    ```

### Build and Push Container Images

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-build-and-push-container-images --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/1.3/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Sample GitHub Actions Workflow](.github/workflows/BuildandPushContainerImage.yml)
- [Sample application with Dockerfile](AKSWebApp)

#### Code Snippets

- (Sample) Set variables used for the sample commands:

    ```bash
    RG=rg-build-and-push-container-images
    ACR=$(az acr list --resource-group $RG --query [].name --output tsv)
    ```

- (Sample) Log in to an Azure Container Registry (ACR):

    ```bash
    az acr login --name $ACR
    ```

- (Sample) Clone a Git repository:

    ```bash
    git clone https://github.com/pluralsight-cloud/aks-deploy-applications
    ```

- (Sample) Build a container image using Docker:

    ```bash
    docker build --tag akswebapp:v1 .
    ```

- (Sample) Tag an image so it can be pushed to an Azure Container Registry (ACR):

    ```bash
    ACR_LOGIN_SERVER=$(az acr show --name $ACR --query loginServer --output tsv)
    docker tag akswebapp:v1 $ACR_LOGIN_SERVER/akswebapp:v1
    ```
- List images with Docker:

    ```bash
    docker image list
    ```

- (Sample) Push a container image to Azure Container Registry (ACR) using Docker:

    ```bash
    ACR_LOGIN_SERVER=$(az acr show --name $ACR --query loginServer --output tsv)
    docker push $ACR_LOGIN_SERVER/akswebapp:v1
    ```

- Build a container image using a quick task in Azure Container Registry (ACR):

    ```bash
    az acr build --registry $ACR --image akswebapp:v1 .
    ```

## Module 2: Deploying Applications

### Customizing Applications for Deployment

#### Resources

- [CI/CD workflow using GitOps (Flux v2)](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/conceptual-gitops-flux2-ci-cd)
- [Tutorial: Deploy applications using GitOps with Flux v2](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/tutorial-use-gitops-flux2?toc=%2Fazure%2Faks%2Ftoc.json&bc=%2Fazure%2Faks%2Fbreadcrumb%2Ftoc.json&tabs=azure-cli)

### Demo: Deploy Applications Manually

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-deploy-applications-manually --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/2.2/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Sample Manifests](Manifests)

#### Code Snippets

- Code Snippet to set commonly used variables and the `k` alias:

    ```bash
    RG=rg-deploy-applications-manually
    AKS=$(az aks list --resource-group $RG --query [].name --output tsv)
    ACR=$(az acr list --resource-group $RG --query [].name --output tsv)
    ACR_LOGIN_SERVER=$(az acr show --name $ACR --query loginServer --output tsv)
    alias k=kubectl
    az aks get-credentials --resource-group $RG --name $AKS
    ```

- Create a namespace and save the config so it can be updated with `k apply`:

    ```bash
    k create namespace akswebapp-prod --save-config
    ```

- (Sample) Generate the YAML for a namespace:

    ```bash
    k create namespace akswebapp-prod --dry-run=client --output yaml > deployment-prod.yaml
    ```

- (Sample) Generate the YAML for a deployment and append it to an existing YAML file:

    ```bash
    k create deployment akswebapp --image=$ACR_LOGIN_SERVER/akswebapp:v1 --namespace akswebapp-prod --replicas=1 --dry-run=client --output yaml >> deployment-prod.yaml
    ```

- (Sample) Generate the YAML for a Load Balancer service and append it to an existing YAML file:

    ```bash
    k create service loadbalancer akswebapp --namespace akswebapp-prod --tcp=80:8080 --dry-run=client --output yaml >> deployment-prod.yaml
    ```

- (Sample) Apply a YAML kubernetes manifest:

    ```bash
    k apply -f deployment-prod.yaml
    ```

- (Sample) List all Kubernetes resources in a namespace:

    ```bash
    k get all --namespace akswebapp-prod
    ```

### Demo: Applying Kustomizations

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-applying-kustomizations --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/2.3/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Sample Kustomizations](Kustomize)
- [Install Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)

#### Code Snippets

- Install Kustomize on Ubuntu using Snap:

    ```bash
    sudo snap install kustomize
    ```

- (Sample) Create a Kustomization:

    ```bash
    kustomize create --autodetect --recursive
    ```

- (Sample) Create an overlay and add resources:

    ```bash
    cd overlays/staging
    kustomize create --namespace staging --namesuffix "-staging" --resources '../../base'
    ```

- (Sample) Build a Kustomization:

    ```bash
    kustomize build akswebapp/overlays/staging
    ```

- (Sample) Apply a Kustomization:

    ```bash
    k apply -k akswebapp/overlays/staging
    ```

### Demo: Deploying Applications with Helm

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-deploying-with-helm --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/2.4/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Sample Chart](Helm)
- [Install Helm](https://helm.sh/docs/intro/install/)
- [Install existing applications with Helm](https://learn.microsoft.com/en-us/azure/aks/kubernetes-helm)

#### Code Snippets

- Install Helm on Ubuntu using Snap:

    ```bash
    sudo snap install helm --classic
    ```

- (Sample) Create an example Helm Chart:

    ```bash
    helm create AKSWebApp
    ```

- (Sample) Install a Helm Chart, creating a namespace, and setting a value:

    ```bash
    helm install akswebapp-staging ./AKSWebApp --namespace akswebapp-staging --create-namespace --set Environment=Staging
    ```

### Automated Application Deployment Methods

- No additional resources.

### Demo: Deploying Applications with Flux

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-deploy-applications-with-flux --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/2.6/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Microsoft Documentation: Deploy applications using GitOps with Flux v2](https://learn.microsoft.com/azure/azure-arc/kubernetes/tutorial-use-gitops-flux2)

#### Code Snippets

- Install the latest `k8s-configuration` and `k8s-extension`.

    ```bash
    az extension add --name k8s-extension
    az extension add --name k8s-configuration
    ```

- (Sample) Create a Flux configuration to enable GitOps on an existing AKS cluster:

    ```bash
    az k8s-configuration flux create \
    --resource-group $RG \
    --cluster-name $AKS \
    --namespace flux \
    --name akswebapp \
    --cluster-type managedClusters \
    --scope cluster \
    --url https://github.com/pluralsight-cloud/aks-deploy-applications/AKSWebApp \
    --branch main  \
    --kustomization name=production path=./Kustomize/akswebapp/overlays/production prune=true \
    --kustomization name=staging path=./Kustomize/akswebapp/overlays/staging prune=true
    ```

### Demo: Deploying Applications with Argo CD

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-deploying-applications-with-argo-cd --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/2.7/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Argo CD - Installation](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- [Argo CD - Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Access The Argo CD API Server](https://argo-cd.readthedocs.io/en/stable/getting_started/#3-access-the-argo-cd-api-server)

#### Code Snippets

- Install Argo CD on Linux:

    ```bash
    VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
    rm argocd-linux-amd64
    ```

- Install Argo CD in your Kubernetes Cluster:

    ```bash
    alias k=kubectl
    k create namespace argocd
    k apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

- Patch the service to update it to be of type Load Balancer:

    ```bash
    k patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    ```

- Retrieve the initial Argo CD admin password:

    ```bash
    argocd admin initial-password -n argocd
    ```

- (Sample) Deploy an application to your AKS Cluster using Argo CD:

    ```bash
    argocd app create akswebapp-staging \
    --repo https://github.com/pluralsight-cloud/aks-deploy-applications \
    --path Helm/AKSWebApp \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace staging \
    --helm-set-string Environment=Staging \
    --sync-option CreateNamespace=true
    ```

    ```bash
    argocd app set akswebapp-prod --sync-policy automated
    ```

- (Sample) Enable auto-sync:

    ```bash
    argocd app set akswebapp-prod --sync-policy automated
    ```

## Module 3: Integrating with Azure Services

### Kubernetes Storage Concepts

- [Container Storage Interface (CSI) drivers on Azure Kubernetes Service (AKS)](https://learn.microsoft.com/azure/aks/csi-storage-drivers)

### Demo: Use Azure Files with AKS

To follow along in this demo using the Cloud Playground Sandbox, follow these steps:

1. Start an [Azure Sandbox](https://app.pluralsight.com/hands-on/playground/cloud-sandboxes).
1. In an InPrivate or Incognito window log in to the Azure Sandbox using the provided credentials.
1. Click the **Deploy to Azure** button. Make sure the link opens in the Sandbox browser tab.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/3.2/main.json)

1. Select the existing **Subscription** and **Resource Group**.
1. Provide the `Application Client ID` and `Secret` from the Sandbox details.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Statically provision a volume](https://learn.microsoft.com/en-us/azure/aks/azure-csi-files-storage-provision#statically-provision-a-volume)
- [Create a custom storage class](https://learn.microsoft.com/en-us/azure/aks/azure-disk-csi#create-a-custom-storage-class)

#### Code Snippets

**pvc.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-azurefiles
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: azurefile-csi
```

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
labels:
    app: pwsh-azurefiles
name: pwsh-azurefiles
spec:
replicas: 2
selector:
    matchLabels:
    app: pwsh-azurefiles
template:
    metadata:
    labels:
        app: pwsh-azurefiles
    spec:
    containers:
    - image: mcr.microsoft.com/powershell
        name: pwsh-azurefiles
        command:
        - "pwsh"
        - "-Command"
        - "while ($true){ Write-Output \"$($env:HOSTNAME): $(Get-Date)\" | Out-File -FilePath 'mnt/azurefiles/date.txt' -Append; Start-Sleep -Seconds 60 }"
        volumeMounts:
        - name: persistent-storage
            mountPath: "/mnt/azurefiles"
            readOnly: false
    volumes:
        - name: persistent-storage
        persistentVolumeClaim:
            claimName: pvc-azurefiles
```

- (Sample) kubectl exec to check the contents of a file:

    ```bash
    k exec deployment/pwsh-azurefiles -- pwsh -Command Get-Content /mnt/azurefiles/date.txt
    ```

- (Sample) Add the Azure Files CSI driver to an existing cluster:

    ```bash
    RG=$(az group list --query [].name --output tsv)
    AKS=$(az aks list --query [].name --output tsv)
    az aks update --name $AKS --resource-group $RG --enable-file-driver
    ```

- List storage classes:

    ```bash
    alias k=kubectl
    k get storageclass
    ```

### AKS Identity Concepts

#### Resources

- [What are workload identities?](https://learn.microsoft.com/entra/workload-id/workload-identities-overview)
- [Workload identity federation](https://learn.microsoft.com/entra/workload-id/workload-identity-federation)
- [Use Microsoft Entra Workload ID with Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview)

### Demo: Configure Workload Identity Federation with AKS

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-configure-workload-identity-federation --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/3.4/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Deploy and configure workload identity on an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/azure/aks/workload-identity-deploy-cluster)

#### Code Snippets

- (Sample) Enable Workload Identity and OIDC issuer on an existing AKS cluster:

    ```bash
    RG=rg-configure-workload-identity-federation
    AKS=$(az aks list --query [].name --resource-group $RG --output tsv)
    az aks update \
    --resource-group $RG \
    --name $AKS \
    --enable-oidc-issuer \
    --enable-workload-identity
    ```

- (Sample) Retrieve OIDC issuer URL for an AKS cluster:

    ```bash
    AKS_OIDC_ISSUER=$(az aks show --name $AKS --resource-group $RG --query "oidcIssuerProfile.issuerUrl" --output tsv)
    ```

- (Sample) Create a managed identity:

    ```bash
    MANAGED_IDENTITY=umi-akswebapp
    LOCATION=eastus
    SUBSCRIPTION=$(az account show --query id --output tsv)
    az identity create \
    --name $MANAGED_IDENTITY \
    --resource-group $RG \
    --location $LOCATION \
    --subscription $SUBSCRIPTION
    ```

- (Sample) Create and annotate a Kubernetes Service Account so it can be projected into a pod and used to retrive a Microsoft Entra token:

    ```bash
    k create serviceaccount sa-akswebapp
    k annotate serviceaccount sa-akswebapp azure.workload.identity/client-id=$MANAGED_IDENTITY_CLIENT_ID
    ```

- (Sample) Create a federated credential:

    ```bash
    az identity federated-credential create \
    --name fed-identity-akswebapp \
    --resource-group $RG \
    --identity-name $MANAGED_IDENTITY \
    --issuer $AKS_OIDC_ISSUER \
    --subject system:serviceaccount:default:sa-akswebapp \
    --audience api://AzureADTokenExchange
    ```

- (Sample) Create an Azure RBAC role assignment for a Workload Identity:

    ```bash
    STORAGE_ID=$(az storage account list --resource-group $RG --query [].id --output tsv)
    MANAGED_IDENTITY_PRINCIPAL_ID=$(az identity show --resource-group $RG --name $MANAGED_IDENTITY --query principalId --output tsv)
    az role assignment create \
    --assignee-object-id $MANAGED_IDENTITY_PRINCIPAL_ID \
    --role 'Storage Blob Data Reader' \
    --scope $STORAGE_ID
    ```

### Demo: Use Azure Key Vault with AKS

1. To follow along with this demonstration you will need your own subscription.
1. Log in to the Azure Portal.
1. Open **Cloud Shell** using **Bash** and set the subscription you'd like to use:

    ```bash
    az account set --subscription "<Subscription ID>"
    ```

    >**Note**: Replace the value of `<Subscription ID>` with the ID of the subscription you'd like to use.

1. Create a resource group for the demonstration.

    > **Note**: You can change the name of the resource group and location as required. But you must use a region where App Gateway for Containers is available.

    ```bash
    RG=$(az group create --location australiaeast --resource-group rg-use-azure-key-vault-with-aks --query name --output tsv)
    ```

1. Click the **Deploy to Azure** button. Make sure the link opens in the same browser tab as the Azure Portal.

    [![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://raw.githubusercontent.com/pluralsight-cloud/aks-deploy-applications/refs/heads/main/Templates/3.5/main.json)

1. Select your preferred **Subscription** and **Resource Group**.
1. Deploy the template.
1. Follow-along with the demo.

#### Resources

- [Deploy and configure workload identity on an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/azure/aks/workload-identity-deploy-cluster)
- [Use the Azure Key Vault provider for Secrets Store CSI Driver in an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/azure/aks/csi-secrets-store-driver)

#### Code Snippets

- (Sample) Enable Key Vault Secrets Store CSI driver on an existing AKS cluster:

    ```bash
    az aks enable-addons \
    --name $AKS \
    --resource-group $RG \
    --addons azure-keyvault-secrets-provider  
    ```

    **secretscore.yaml**
    ```yaml
    apiVersion: secrets-store.csi.x-k8s.io/v1
    kind: SecretProviderClass
    metadata:
    name: # Update with Key Vault name
    spec:
    provider: azure
    parameters:
        usePodIdentity: "false"
        clientID: # Update
        keyvaultName: # Update
        objects:  |
        array:
            - |
            objectName: #Update             
            objectType: secret              
        tenantId: # Update
    ```

- (Sample) Retrieve the required values to populate `secretstore.yaml`:

    ```bash
    echo clientID: $(az identity show --resource-group $RG --name $MANAGED_IDENTITY --query 'clientId' -o tsv)
    echo keyvaultName: $(az keyvault list --resource-group $RG --query [].name --output tsv)
    echo objectName: $(az keyvault secret list --vault-name $KV --query [].name --output tsv)
    echo tenantId: $(az keyvault show --name $KV --resource-group $RG --query properties.tenantId -o tsv)
    ```
