# CloudNativePG High-Availability PostgreSQL Deployment

This repository contains the manifest file (`ha-cluster.yml`) needed to deploy a highly available PostgreSQL cluster on a Kubernetes cluster. The deployment is managed by the CloudNativePG (CNPG) Operator, which is installed using Helm.

## Prerequisites

*   A Kubernetes cluster (e.g., KinD, minikube, GKE, EKS, AKS).
*   The `kubectl` command-line tool configured for your cluster.
*   The `helm` command-line tool installed.

## Deployment Steps

### Step 1: Install the CloudNativePG Operator
First, install the CNPG operator on your cluster using Helm.

```sh
# Add the CloudNativePG Helm repository
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update

# Install the CNPG operator into the 'cnpg-system' namespace
helm upgrade --install cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg

Verify that the operator pod is running: 
kubectl get pods -n cnpg-system

Step 2: Deploy the PostgreSQL Cluster
Apply the ha-cluster.yml manifest to create a high-availability PostgreSQL cluster with three instances.
kubectl apply -f ha-cluster.yml

Monitor the pods until all three pg-ha-cluster instances are running:
kubectl get pods -w

Step 3: Access the Database
The CNPG operator automatically creates and manages credentials in Kubernetes Secrets and provides services for cluster access.
How to Get Credentials
The credentials are Base64 encoded for security. Use the following commands to retrieve the plain-text credentials for the default application user.
Get the application username:
kubectl get secret pg-ha-cluster-app -o=jsonpath='{.data.username}' | base64 --decode

Get the application password:
kubectl get secret pg-ha-cluster-app -o=jsonpath='{.data.password}' | base64 --decode

How to Connect with psql

    Forward the read/write service port in a terminal:
    sh

    kubectl port-forward svc/pg-ha-cluster-rw 5432:5432

    Use code with caution.

Keep this terminal window open.
Connect with psql in another terminal, using the decoded credentials.
sh

psql -h localhost -p 5432 -U <app_user_name> -d <app_user_name>

Use code with caution.

Cleanup
To remove the PostgreSQL cluster and the operator:
sh

kubectl delete -f ha-cluster.yml
helm uninstall cnpg --namespace cnpg-system

