---

# Consul Deployment on Kubernetes

This repository contains the Kubernetes deployment configuration for HashiCorp **Consul** using **Helm**. The deployment includes:

* A 3-node Consul server cluster
* Consul clients with Connect enabled
* Web UI enabled for monitoring and configuration
* Connect injector and controller for service mesh support

## Prerequisites

* Kubernetes cluster (tested on **Kind / Minikube / AKS / GKE**)
* Helm 3 installed
* `kubectl` configured to access your cluster

## Installation Steps

1. **Add the HashiCorp Helm repository**

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```

2. **Deploy Consul using Helm**

```bash
helm install consul hashicorp/consul \
  --values config.yml \
  --create-namespace \
  --namespace consul \
  --version "1.8.1"
```

> `config.yml` is already included in the repo with the necessary configuration for servers, clients, UI, and Connect injection.

3. **Verify the deployment**

```bash
kubectl get pods -n consul
kubectl get svc -n consul
```

* Pods like `consul-server-0`, `consul-connect-injector`, and `consul-webhook-cert-manager` should be in **Running** state.
* The `consul-server` service should be of type **ClusterIP**.

4. **Access the Consul Web UI**

Port-forward the service to your local machine:

```bash
kubectl port-forward svc/consul-ui 8500:http -n consul
```

Then open your browser and go to:

```
http://localhost:8500
```

You will be able to:

* View cluster members and health
* Manage Key/Value entries
* Inspect registered services
* Use Connect service mesh features

## Notes

* If you face `CrashLoopBackOff` on the injector, ensure that the **server pods are fully initialized**.
* PodSecurityPolicies are disabled (`enablePodSecurityPolicies: false`) for compatibility with Kubernetes v1.27+.
* No enterprise license is required for this setup.

---

