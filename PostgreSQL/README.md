-----

# Deploying a High-Availability PostgreSQL Cluster on Kubernetes

This guide provides a step-by-step tutorial for deploying a robust, 3-node PostgreSQL cluster on Kubernetes. It features primary-replica replication and automatic failover, managed by the powerful and easy-to-use **CloudNativePG Operator**.

This setup is ideal for creating a production-grade database foundation for any stateful application.

## Prerequisites

Before you begin, you will need:

  * A running Kubernetes cluster. For local development, **[Kind](https://kind.sigs.k8s.io/)** is recommended.
  * `kubectl` installed and configured to communicate with your cluster.

> **Note for Kind Users**: If you are using a recent version of Kind, it's recommended to create your cluster with a special configuration to avoid potential security profile issues with the operator's webhook. An example `kind-config.yaml` should be provided in your repository.
>
> Create the cluster with: `kind create cluster --config kind-config.yaml`

## Deployment Steps

This guide assumes all necessary YAML files (`postgres-namespace.yml`, `postgres-cluster.yaml`) are present in this repository.

### 1\. Create a Namespace

This command uses the `postgres-namespace.yml` file to create a dedicated, isolated workspace for all database resources.

```bash
kubectl apply -f postgres-namespace.yml
```

### 2\. Install the CloudNativePG Operator

First, download the operator manifest from its official GitHub repository to a local file. Then, apply the local file.

1.  **Download the manifest:**
    ```bash
    curl -L -o cnpg-operator.yaml https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/main/releases/cnpg-1.22.1.yaml
    ```
2.  **Apply the local file:**
    ```bash
    kubectl apply -f cnpg-operator.yaml
    ```
3.  **Wait for the operator to be ready.** You can watch its status until the pod is `1/1 Running`:
    ```bash
    kubectl get pods -n cnpg-system --watch
    ```

### 3\. Deploy the PostgreSQL Cluster

This command reads the `postgres-cluster.yaml` manifest and tells the operator to build your 3-node database cluster.

```bash
kubectl apply -f postgres-cluster.yaml
```

## Management and Verification

### Check Cluster Status

You can check the status of your newly created cluster. It will take a few minutes to bootstrap.

```bash
kubectl get cluster -n postgres-ns --watch
```

Wait for the `STATUS` to show `Cluster in healthy state`.

### Identify the Primary (Master) Pod

The operator automatically labels the pods with their roles. You can find the current primary pod with this command:

```bash
kubectl get pods -n postgres-ns -l cnpg.io/role=primary
```

### Test Automatic Failover

You can simulate a failure by deleting the primary pod. The operator will detect this and automatically promote one of the replicas to be the new primary.

1.  Get the name of the current primary pod (e.g., `ca-postgres-cluster-1`).
2.  Delete it:
    ```bash
    kubectl delete pod ca-postgres-cluster-1 -n postgres-ns
    ```
3.  Watch the pods. You will see the old pod terminate and a new one get created.
    ```bash
    kubectl get pods -n postgres-ns --watch
    ```
4.  After a minute, check for the new primary. A different pod will now have the `primary` label.
    ```bash
    kubectl get pods -n postgres-ns -l cnpg.io/role=primary
    ```

## Accessing the Database

The operator automatically creates `Services` for network access and a `Secret` to hold the credentials.

1.  **Find the Service Endpoints**:

    ```bash
    kubectl get service -n postgres-ns
    ```

      * The service ending in **`-rw`** (e.g., `ca-postgres-cluster-rw`) is the read-write endpoint that always points to the primary pod. Your applications should use this as their database host.
      * The service ending in **`-ro`** is the read-only endpoint that load balances across the replica pods.

2.  **Find the User Secret**: The operator creates a secret for the database user defined in your `postgres-cluster.yaml`. It is named `<cluster-name>-<user-name>`.

    ```bash
    # The secret will be named 'ca-postgres-cluster-caadmin'
    kubectl get secret -n postgres-ns
    ```

3.  **Retrieve the Password**: To get the password for your application, you can decode it from the secret:

    ```bash
    # Replace the secret name with the one from the command above
    kubectl get secret ca-postgres-cluster-caadmin -n postgres-ns -o jsonpath='{.data.password}' | base64 --decode
    ```

## Cleanup

To delete all the resources created in this guide, run the following commands in order:

```bash
# 1. Delete the PostgreSQL cluster
kubectl delete -f postgres-cluster.yaml

# 2. Delete the namespace
kubectl delete -f postgres-namespace.yml

# 3. Uninstall the operator using the local file
kubectl delete -f cnpg-operator.yaml
```
