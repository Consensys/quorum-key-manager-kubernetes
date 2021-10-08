# Quorum Key Manager Kubernetes

This repository contains an implementation example on how to deploy the Quorum Key Manager with Hashicorp Vault using Kubernetes and Helm charts.

# 1. Installing Quorum Key Manager

## 1.1. Quickstart

1. To deploy a simple Quorum Key Manager (not production ready) and with Hashicorp Vault, run the following command:

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
# 2. Hashicorp Vault

## 2.1 Enable auditing

Once vault auditing has been enabled, you will need to activate it by running the following command

```bash
kubectl exec -ti $POD NAME --  vault audit enable file file_path=/vault/audit/vault_audit.log
```

# 3. TLS configuration

TLS activation requires that you are able to define secrets and retrieve certificates values.

Secrets are k8s secrets and are used to hold the servers TLS material.
Certificate values, which are on the client side, must be set in this chart values.

Below are given commands and instructions to help you creating your own secrets with the valid names and setting values.

(this procedure is not applicable if you use existing secrets, as the ones given by a cert-manager for instance)


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


## 3.1 Vault tls

Create a server secret (optional)

Secret name must be `vault-tls`

```bash
kubectl create secret generic vault-tls --from-file=ca.crt=$VAULT_CA_FILE --from-file=tls.crt=$VAULT_SERVER_CRT_FILE --from-file=tls.key=$VAULT_SERVER_KEY_FILE -n qa-qkm
```

### client certificate

fill the `vaultAgent` part in the file below.

[here](./values/qkm.yaml.gotmpl)

format is base64 pem encoded file for each value

# 4. Vault node

It is recommanded when running in production to have your Vault instance isolated from other processes.

To help you achieve that isolation we have added a `nodeSelector` option that will identify the dedicated node within your cluster.

Run the following command to mark the targeted/dedicated node

```bash
kubectl label node $VAULT_DEDICATED_NODE_ID type=`nodeSelector`
```

For k8s clusters with versions >= 1.18 the following command will also help exclude non Vault processes

```bash
kubectl taint nodes $VAULT_DEDICATED_NODE_ID isVaultDedicated=true:NoSchedule
```

This exclusion policy is permitted by the use of k8s `tolerations`, when no exclusion is necessary the above command must be omitted or cancelled like so :

```bash
kubectl taint nodes $VAULT_DEDICATED_NODE_ID isVaultDedicated=true:NoSchedule-
```

and the `dedicatedNode` flag, in values, must be set to false ( that option is not recommanded )
