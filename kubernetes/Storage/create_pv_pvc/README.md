COURSE:        Storage for CKA
AUTHOR:        Antonio Jesus Piedra
DISCLAIMER:    The scripts and configuration files used in this course have been
               tested in macOS. If used in other platform, some of the steps may 
               need to be adjusted briefly but the concepts will remain the same.
PREREQUISITES: The following command line tools will be usefull during the course:
                * vim
				    * minikuke
                * kubectl
______________________________________


# Cluster Setup Instructions

## Prerequisites

1. Download & install Docker Desktop from [Docker's official website](https://www.docker.com/products/docker-desktop)

2. Install Minikube: `brew install minikube`


## Starting the Cluster

1. Start the Minikube cluster with Docker driver:
   ```bash
   minikube start --driver=docker --nodes 3
   ```

2. Verify cluster is running:
   ```bash
   kubectl cluster-info
   ```

## Setting up Course Namespaces

1. Create all required namespaces:
   ```bash
   # Create namespaces for all modules and mock exams
   kubectl create namespace module1
   kubectl create namespace module2
   kubectl create namespace module3
   kubectl create namespace mock-exam1
   kubectl create namespace mock-exam2

   # Verify namespaces were created
   kubectl get namespaces
   ```

2. Create contexts for each namespace:
   ```bash
   # Create contexts
   kubectl config set-context module1-context --cluster=minikube --user=minikube --namespace=module1
   kubectl config set-context module2-context --cluster=minikube --user=minikube --namespace=module2
   kubectl config set-context module3-context --cluster=minikube --user=minikube --namespace=module3
   kubectl config set-context mock-exam1-context --cluster=minikube --user=minikube --namespace=mock-exam1
   kubectl config set-context mock-exam2-context --cluster=minikube --user=minikube --namespace=mock-exam2

   # Verify contexts were created
   kubectl config get-contexts
   ```

3. Switch between contexts:
   ```bash
   # Example: Switch to module1 context
   kubectl config use-context module1-context

   # Verify current context
   kubectl config current-context
   ```

4. Grant the necessary RBAC permissions to the storage-provisioner service account:
   ```bash
   kubectl create clusterrolebinding storage-provisioner-node-binding --clusterrole=system:node --serviceaccount=kube-system:storage-provisioner
   ```


## Stopping the Cluster

When you're done working:
```bash
minikube stop
```

To delete the cluster completely:
```bash
minikube delete --purge
```
