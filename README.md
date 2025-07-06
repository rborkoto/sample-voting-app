# Sample Voting App

A simple distributed application running across multiple containers forked from dockersamples. This fork has some updated readme guidelines to help with deployment across any platform

## Prerequisites

Before you begin, ensure you have:
- `kubectl` installed and configured for your target cluster
- Access and credentials for each Kubernetes platform (Minikube, EKS, GKE, AKS)
- The `k8s-specifications/` directory from this repo, containing all YAML manifests

## Deployment Instructions

Below are instructions to deploy the Voting App solely via Kubernetes. All steps assume you are in the root of the cloned repo.

### Minikube

1. Start Minikube with necessary addons:
   ```shell
   minikube start --addons=ingress
   ```
2. Deploy the app:
   ```shell
   kubectl create -f k8s-specifications/
   ```
3. (Optional) Expose Ingress on localhost:
   ```shell
   minikube tunnel
   ```
4. Access the services:
   ```shell
   minikube service vote   -n voting-app --url   # prints the Vote UI URL
   minikube service result -n voting-app --url   # prints the Result UI URL
   ```
5. To clean up(once done):
   ```shell
   kubectl delete -f k8s-specifications/
   ```

---

### Amazon EKS

1. Install & configure `eksctl` and AWS CLI, and authenticate:
   ```shell
   aws configure
   eksctl create cluster --name vote-cluster --region <region> --nodes 2
   ```
2. Ensure `kubectl` context is set to the new cluster:
   ```shell
   aws eks --region <region> update-kubeconfig --name vote-cluster
   ```
3. Deploy Voting App to EKS:
   ```shell
   kubectl create namespace voting-app
   kubectl create -f k8s-specifications/ -n voting-app
   ```
4. Expose with LoadBalancer or NodePort (depending on your setup):
   - If using NodePort:
     ```shell
     kubectl get svc -n voting-app
     # Note the EXTERNAL-IP or NodePort
     ```
   - If using Ingress + ALB ingress controller, apply your Ingress manifest.

5. Clean up(once done):
   ```shell
   kubectl delete -f k8s-specifications/ -n voting-app
   eksctl delete cluster --name vote-cluster --region <region>
   ```

---

### Google Kubernetes Engine (GKE)

1. Install & configure `gcloud` CLI, then authenticate and set project:
   ```shell
   gcloud auth login
   gcloud config set project <PROJECT_ID>
   ```
2. Create a GKE cluster:
   ```shell
   gcloud container clusters create vote-cluster --zone <zone>
   ```
3. Get cluster credentials:
   ```shell
   gcloud container clusters get-credentials vote-cluster --zone <zone>
   ```
4. Deploy the app:
   ```shell
   kubectl create namespace voting-app
   kubectl create -f k8s-specifications/ -n voting-app
   ```
5. Access services:
   - For `NodePort` services, run:
     ```shell
     kubectl get svc -n voting-app
     ```
   - For `LoadBalancer` services, note the `EXTERNAL-IP`.

6. Tear down(once done):
   ```shell
   kubectl delete -f k8s-specifications/ -n voting-app
   gcloud container clusters delete vote-cluster --zone <zone>
   ```

---

### Azure Kubernetes Service (AKS)

1. Install & configure Azure CLI, then login:
   ```shell
   az login
   az account set --subscription <SUBSCRIPTION_ID>
   ```
2. Create an AKS cluster:
   ```shell
   az aks create --resource-group vote-rg --name vote-aks --node-count 2 --generate-ssh-keys
   ```
3. Get credentials:
   ```shell
   az aks get-credentials --resource-group vote-rg --name vote-aks
   ```
4. Deploy the app:
   ```shell
   kubectl create namespace voting-app
   kubectl create -f k8s-specifications/ -n voting-app
   ```
5. Access the services:
   ```shell
   kubectl get svc -n voting-app
   # Use EXTERNAL-IP for LoadBalancer or NodePort as needed
   ```
6. Clean up (once done):
   ```shell
   kubectl delete -f k8s-specifications/ -n voting-app
   az aks delete --resource-group vote-rg --name vote-aks --yes

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The simple voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

We are not generating any mock traffic to keep it lightweight. Feel free to use any tool to generate traffic or manually create traffic. 

More details on the required [Datadog implementation requirements here](https://github.com/rborkoto/sample-voting-app/blob/main/datadog-implementation/readme.md)
