# Quorum Key Manager Kubernetes

This repository contains an implementation example on how to deploy the Quorum Key Manager using Kubernetes and Helm charts.

# 1. Installing Quorum Key Manager

## 1.1. Prerequisites, recommendations

In order to simply help you filling your env vars it is recommended that you have the `envsubst`utility installed on your platform.

You should also have all the necessary tools to run helmfile properly.

- Up to date kubectl
- Up to date helm and helm diff plugin

## 1.2. Quickstart

1. To deploy Quorum Key Manager solution, do the following:


```bash
# Adapt the manifests.yaml with your own needs and values
# then build up the B64_MANIFESTS like so
export B64_MANIFESTS=$(envsubst < manifests/manifests.yaml | base64)

# Adapt your env variables accordingly
cp .env.sample .env
# then
export $(xargs < .env)
# finally
helmfile apply --suppress-secrets
```

You may find the nodes, roles and stores manifests definition in the dedicated section [here](./manifests/maifests.yaml) and then adapt it to yours needs following instructions found in [documentation](https://docs.quorum-key-manager.consensys.net)

2. Once deployed you could easily test the Quorum Key Manager API in http://localhost:8080:

```
export QKM_POD_NAME=$(kubectl get pods --namespace $YOUR_NS -l "app.kubernetes.io/name=quorumkeymanager,app.kubernetes.io/instance=quorum-key-manager" -o
kubectl port-forward --namespace $YOUR_NS $QKM_POD_NAME 8080:8080
```

[See Quorum Key Manager APIs documentation](https://consensys.github.io/quorum-key-manager)

## 1.3. Delete Quorum Key Manager
!!!hint
  to delete Quorum Key Manager's deployment and its ressources run the following commands:

```bash
helmfile delete --purge
kubectl delete namespace qa-qkm
```

# 2. TLS configuration

TLS activation requires that you are able to define secrets and retrieve certificates values.

Secrets are k8s secrets and are used to hold the servers TLS material.
Certificate values, which are on the client side, must be set in this chart values.

Below are given commands and instructions to help you creating your own secrets with the valid names and setting values.

(this procedure is not applicable if you use existing secrets, as the ones given by a cert-manager for instance)

## 2.1. Quorum Key Manager tls

The server TLS configuration should be located in an *existing secret* pointed to by the `auth.tls.secretName` value found in the charts

## 2.2. Postgres tls

### server certificate

Create a server secret (optional)

Secret name must be `postgres-certificates-tls-secret`

```bash
kubectl create secret generic postgres-certificates-tls-secret --from-file=$PGSERVER_CRT_FILE --from-file=$PGSERVER_KEY_FILE --from-file=$PGCA_CERTS_FILE -n qa-qkm
```

# 3. Manifests

Manifests values must be passed to the chart using the `B64_MANIFESTS` env var. This env var is required.

Using the provided manifest sample in the manifests dir you can build this value using the following command

```
export B64_MANIFESTS=$(envsubst < manifests/manifests.yaml | base64)
```

Please note that you will need `envsubst` tool installed and that you should fill related env vars accordingly to fit your needs.

[See Quorum Key Manager APIs documentation](https://consensys.github.io/quorum-key-manager) for manifests building.

# 4. Env vars

In the `env.sample` file you you find a sample of the env vars you need to set with your own values.

It is recommanded that you make your own copy of this file to `.env` and then run something similar to 

```
export $(xargs < .env)
```

Other env vars are involved in the manifest creation process as seen above, these are left to your discretion.


test
