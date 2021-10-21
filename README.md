# Quorum Key Manager Kubernetes

This repository contains an implementation example on how to deploy the Quorum Key Manager using Kubernetes and Helm charts.

# 1. Installing Quorum Key Manager

## 1.1. Quickstart

1. To deploy a simple Quorum Key Manager (not production ready), run the following command:

```bash
helmfile apply --suppress-secrets
```

You may find the stores manifests definition in the dedicated section [here](./values/qkm.yaml.gotmpl) and then adapt it to yours needs following instructions found in doc

2. Once deployed you could easily test the Quorum Key Manager API in http://localhost:8080:

```
kubectl port-forward --namespace qa-qkm svc/quorum-key-manager-quorumkeymanager 8080:8080
```

[See Quorum Key Manager APIs documentation](https://consensys.github.io/quorum-key-manager)

## 1.2. Delete Quorum Key Manager
!!!hint
  to delete Quorum Key Manager's deployment and its ressources run the following commands:

```bash
helmfile delete --purge
kubectl delete namespace qa-qkm
```

# 3. TLS configuration

TLS activation requires that you are able to define secrets and retrieve certificates values.

Secrets are k8s secrets and are used to hold the servers TLS material.
Certificate values, which are on the client side, must be set in this chart values.

Below are given commands and instructions to help you creating your own secrets with the valid names and setting values.

(this procedure is not applicable if you use existing secrets, as the ones given by a cert-manager for instance)

## 3.1 Quorum Key Manager tls

The server TLS configuration should be located in an existing secret pointed to by the `auth.tls.secretName` value found in the charts

## 3.1 Postgres tls

### server certificate

Create a server secret (optional)

Secret name must be `postgres-certificates-tls-secret`

```bash
kubectl create secret generic postgres-certificates-tls-secret --from-file=$PGSERVER_CRT_FILE --from-file=$PGSERVER_KEY_FILE --from-file=$PGCA_CERTS_FILE -n qa-qkm
```

### client certificate

fill the `postgresql` part in the file below.

[here](./values/qkm.yaml.gotmpl)

format is base64 pem encoded file for each value

# 4 Manifests

Manifests values must be passed to the chart using the `B64_MANIFESTS` env var. This env var is required. 