# Quorum Key Manager Kubernetes

This repository contains an implementation example on how to deploy the Quorum Key Manager with Hashicorp Vault using Kubernetes and Helm charts.

# 2. Installing Quorum Key Manager

## 2.1. Quickstart

1. To deploy a simple Quorum Key Manager (not production ready) and with Hashicorp Vault, run the following command:

```bash
helmfile apply --suppress-secrets
```

2. Once deployed you could easily test the Quorum Key Manager API in http://localhost:8080:

```
kubectl port-forward --namespace staging-qkm svc/quorum-key-manager-quorumkeymanager 8080:8080
```

[See Quorum Key Manager APIs documentation](https://consensys.github.io/quorum-key-manager)

## 2.2. Delete Quorum Key Manager
!!!hint
  to delete Quorum Key Manager's deployment and its ressources run the following commands:

```bash
helmfile delete --purge
kubectl delete namespace staging-qkm
```
