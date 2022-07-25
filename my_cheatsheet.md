# My Cheat Sheet

## DOCKER

### How to create Dockerfile

**Dockerfile example for myinsuranceapp**

```
FROM ubuntu
RUN apt-get update && apt-get install -y python3 python3-pip
COPY . /
RUN pip3 install -r requirements.txt
RUN python3 project/init/init_db.py
# WORKDIR myinsuranceapp
CMD ["python3", "runserver.py"]
EXPOSE 5000
```
### How to build a Docker image
- docker build -t <image_name:name_tag> -f Dockerfile .
### To run image in container
- docker run -d <image_name:name_tag>
### To stop container
- docker container stop <container_id>
### To delete stopped container
- docker prune
### To run the server in the background
- docker run -d -p <portnumber:portnumber>
### To test the server response, please open a new tab in cmder
- curl http://localhost:port_number/login
### To change the name of the image
- docker tag <current_image_name:name_tag> <new_image_name:name_tag>
### To push docker image
- docker push <image_name:name_tag>
### To see Jenkins status
- sudo service jenkins status
### If jenkins status is not active, to activate it
- sudo service Jenkins start
### Add jenkins permission 
- sudo usermod -a -G docker jenkins
- restart jenkins

[Jenkins Script for Docker](https://github.com/ricardoahumada/python-package-flask-test/blob/docker/Jenkinsfile-docker)

**Don't forget to stop the running server**

## KUBERNETES

### How to create yaml file for deployment

**Deployment.yaml example for myinsuranceapp**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myinsuranceapp-v1-deployment
  labels:
    app: myinsuranceapp-v1
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myinsuranceapp-v1
  template:
    metadata:
      labels:
        app: myinsuranceapp-v1
    spec:
      containers:
      - name: myinsuranceapp-v1
        image: duygugurbuzyildiz/myinsuranceapp-v1:v1.2
        ports:
        - name: http
          containerPort: 5000
```
**Service.yaml example for myinsuranceapp**
```
apiVersion: v1
kind: Service
metadata:
  name: myinsuranceapp-v1
  labels:
    app: myinsuranceapp-v1
spec:
  selector:
    app: myinsuranceapp-v1
  type: NodePort
  ports:
  - port: 5000
    targetPort: http
    # nodePort: 31012
    protocol: TCP
    name: http
```
### Execute Normal
**Check status of minikube**
- minikube status

**If it doesn't running, start minikube**

- minikube start

    - kubectl apply -f <deployment_yaml_file_name>.yaml
    #### To get deployment and service name
    - kubectl get all
    - kubectl logs deployment/<deployment_name>
    - kubectl apply -f <service_yaml_file_name>.yaml
    #### To get deployment and service name
    - kubectl get all
    - kubectl port-forward service/<service_name> <portnumber:portnumber>
- to check consume of api, open a new tab in cmder
    - curl localhost:5000/api/v1/restricted
- to delete 
    - kubectl delete service/<service_name> deployment/<deployment_name>

**Azure Deployment.yaml example for myinsuranceapp**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myinsuranceapp-v1-deployment-az
  labels:
    app: myinsuranceapp-v1-az
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myinsuranceapp-v1-az
  template:
    metadata:
      labels:
        app: myinsuranceapp-v1-az
    spec:
      imagePullSecrets:
      - name: acr-secret
      containers:
      - name: myinsuranceapp-v1-az
        image: duygubootcampregistry.azurecr.io/myinsurance-v1:v1.2
        ports:
        - name: http
          containerPort: 5000
```
**Azure Service.yaml example for myinsuranceapp**
```
apiVersion: v1
kind: Service
metadata:
  name: myinsuranceapp-v1-service-az
  labels:
    app: myinsuranceapp-v1-az
spec:
  type: NodePort
  selector:
    app: myinsuranceapp-v1-az
  ports:
  - port: 5000
    targetPort: http
    # nodePort: 30564
    protocol: TCP
    name: http
```
### Execute AZ Normal
**Create an azure kubernetes cluster manually**

- Needs AZ installation
    - curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    - az login
    - az aks get-credentials --resource-group [myResourceGroup] --name [myAKSCluster] --admin
    - kubectl config get-contexts
    - kubectl config set current-context <context_name>
    - kubectl get nodes

- Login into azure registry:

    - docker login [acr-path]
- Add credentials to kubectl:
    - kubectl create secret generic acr-secret --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson
- kubectl apply -f <azure_deployment_yaml_file>
- kubectl apply -f <azure_service_yaml_file>
- kubectl port-forward service/<service_name_azure> <port_number:por_number>

- Clean up:
    - az logout
    - kubectl config set current-context minikube

-To create a jenkins pipeline
    - Need to create credentials named "k8-credentials"  uploading minikube config file.

    **open a new cmder tab and use: "scp ubuntu@[IP]:.kube/config k8-config" to download config file with name k8-config in your windows virtual machine**

    **For getting the server url use: "az aks list -o table" and use "Fqdn" value**
        
        - withKubeConfig([credentialsId: 'k8-credentials',serverUrl:'https:<az_aks_list_name>']) {
        sh 'kubectl apply -f folder_name/deploy_yaml_file'
        sh 'kubectl apply -f folder_name/service_yaml_file'
        sh 'kubectl port-forward service/service_name port_name:port_name &'

[Jenkins Script for Kubernetes](https://github.com/ricardoahumada/python-package-flask-test/blob/kubernetes/Jenkinsfile-k8)

## TERRAFORM

### AZ install
- curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
- az login
- az account list
- az vm list

- To initialize terraform
    - terraform init
- To create a principal
    - az ad sp create-for-rbac --skip-assignment
    - Copy and paste the appId and password in a text file
    - Convert them as a variable with;
        - export TF_VAR_APPID=<appID_value>
        - export TF_VAR_PASSWORD=<password_valu>
    - Check the variables 
        - echo $TF_VAR_APPID
        - echo $TF_VAR_PASSWORD
- Create .tf files

**aks-cluster.tf**
```
resource "random_pet" "prefix" {}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "default" {
  name     = "${random_pet.prefix.id}-rg"
  location = "eastasia"

  tags = {
    environment = "Demo"
  }
}

resource "azurerm_kubernetes_cluster" "default" {
  name                = "${random_pet.prefix.id}-aks"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  dns_prefix          = "${random_pet.prefix.id}-k8s"

  default_node_pool {
    name            = "default"
    node_count      = 1
    vm_size         = "Standard_D2_v2"
    os_disk_size_gb = 30
  }

  service_principal {
    client_id     = var.APPID
    client_secret = var.PASSWORD
  }

  role_based_access_control {
    enabled = true
  }

  tags = {
    environment = "Demo"
  }
}
```
**outputs.tf**
```
output "resource_group_name" {
  value = azurerm_resource_group.default.name
}

output "kubernetes_cluster_name" {
  value = azurerm_kubernetes_cluster.default.name
}
```

**variables.tf**
```
variable "APPID" {
  description = "Azure Kubernetes Service Cluster service principal"
}

variable "PASSWORD" {
  description = "Azure Kubernetes Service Cluster password"
}
```

**versions.tf**
```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.66.0"
    }
  }

  required_version = ">= 0.14"
}
```
### validate terraform
- terraform validate
### execute terraform
- terraform apply

after terraform apply, check the azure account to see the cluster.

to get credentials from new cluster use command below:
- az aks get-credentials --resource-group $(terraform output -raw resource_group_name) --name $(terraform output -raw kubernetes_cluster_name)
### download kube config to your laptop
- open a new tab in cmder and download k8-config document in your windows virtual machine with the command below
    - scp ubuntu@[ubuntu IP]:.kube/config k8-config
    - kubectl config get-contexts
    - kubectl config set current-context <context_name>

to execute Jenkins file, we are adding kubectl port-forward command into our jenkins file

 - withKubeConfig([credentialsId: 'k8-credentials',serverUrl:'https:<az_aks_list_name>']) {
        sh 'kubectl apply -f folder_name/deploy_yaml_file'
        sh 'kubectl apply -f folder_name/service_yaml_file'
        sh 'kubectl port-forward service/service_name port_name:port_name &'



[Jenkins Script for Terraform](https://github.com/ricardoahumada/terraform/blob/master/jenkins/Jenkinsfile-terraform)

## Acceptance Test

### How do we test it?
- For testing this, we need to send requests to the endpoint.
- We are going to need to send a token, because is a "restricted" area. 
- So we need to get a token first (in a concrete function).
- Then, we can send a request with the token and evaluate the response, verifying that we receive the response code 200 and a list of products (another test function).
- We can test too invalida situations, like sending invalid/fake tokens or requesting products for non-existing users (e.g. user 34567, doesn't exist in database)
- Tests steps
    - First, we need a function to get the token: since the endpoint is restricted area.
    - Then, a function for sending VALID requests to the endpoint, using the valid token. - If you want you can test various valid scenarios in various functions.
    - Finally, is a good idea to create a function for sending INVALID requests to the endpoint.
        - For example sending invalid or fake tokens
        - Requesting non-existing users etc.
        - you can use various functions for this
### Two types of tests
- We can use two types of tokens for testing the endpoint:

    - Using the "flask test_client": to simulate the requests.
        - For this, we need to import the app in the test case.
        - We DON'T need the app running for real.
        - You can see these tests in: [tests/acceptance-flask](https://github.com/ricardoahumada/myinsuranceapp/blob/main/tests/acceptance-flask/test_app_flask_tester.py)
    - Using the python "request" library: for sending real requests to the endpoint.
        - For this we DO need the app be running (executing: python3 runserver.py)
        - You can see these tests in: [tests/acceptance-request](https://github.com/ricardoahumada/myinsuranceapp/blob/main/tests/acceptance-request/test_app_request.py)


