
---

#  CloudNativePG High-Availability PostgreSQL Deployment

This repository contains the manifest file (**`ha-cluster.yml`**) required to deploy a **highly available PostgreSQL cluster** on Kubernetes.
The deployment is managed by the **CloudNativePG (CNPG) Operator**, installed via **Helm**.

---

##  Prerequisites

Before you begin, ensure you have the following:

* A running **Kubernetes cluster** (KinD, Minikube, GKE, EKS, or AKS)
* The **`kubectl`** CLI tool configured for your cluster
* The **`helm`** CLI tool installed

---

##  Deployment Steps

### **Step 1: Install the CloudNativePG Operator**

First, install the CNPG Operator on your cluster using Helm.

```sh
# Add the CloudNativePG Helm repository
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update

# Install the CNPG operator into the 'cnpg-system' namespace
helm upgrade --install cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg
```

 **Verify that the operator pod is running:**

```sh
kubectl get pods -n cnpg-system
```

---

### **Step 2: Deploy the PostgreSQL Cluster**

Apply the `ha-cluster.yml` manifest to create a **high-availability PostgreSQL cluster** with three instances.

```sh
kubectl apply -f ha-cluster.yml
```

 **Monitor the pods until all three pg-ha-cluster instances are running:**

```sh
kubectl get pods -w
```

---

### **Step 3: Access the Database**

The CNPG operator automatically:

* Creates and manages **credentials** in Kubernetes Secrets.
* Provides **services** for accessing the PostgreSQL cluster.

---

####  How to Get Credentials

The credentials are **Base64 encoded** for security.
Use the following commands to retrieve the plain-text credentials for the default application user.

**Get the application username:**

```sh
kubectl get secret pg-ha-cluster-app -o=jsonpath='{.data.username}' | base64 --decode
```

**Get the application password:**

```sh
kubectl get secret pg-ha-cluster-app -o=jsonpath='{.data.password}' | base64 --decode
```

---

####  How to Connect with `psql`

1. **Forward the read/write service port in a terminal:**

   ```sh
   kubectl port-forward svc/pg-ha-cluster-rw 5432:5432
   ```

   Keep this terminal window open.

2. **Connect with `psql` in another terminal**, using the decoded credentials:

   ```sh
   psql -h localhost -p 5432 -U <app_user_name> -d <app_user_name>
   ```

---

##  Cleanup

To remove both the PostgreSQL cluster and the CNPG operator:

```sh
kubectl delete -f ha-cluster.yml
helm uninstall cnpg --namespace cnpg-system
```

---

###  Summary

You have now successfully:

* Installed the CloudNativePG Operator
* Deployed a 3-node HA PostgreSQL cluster
* Retrieved credentials and connected via `psql`

---

