# ExternalSecrets + HashiCorp Vault on K3D (easy install)

This repo lets you install and set up [HashiCorp Vault](https://www.vaultproject.io)
and the [ExternalSecrets](https://external-secrets.io/v0.7.2/api/externalsecret/)
operator on K3D (or another Kubernetes instance) easily.
It does so using [helmfile](https://helmfile.readthedocs.io/).

**IMPORTANT**: this will install the Vault in "dev mode". This is highly
insecure but allows you to quickly experiment with the technologies.
Do not use it in production!

## What this repo does exactly


* Deploy a Vault in the namespace `vault`, with a root access token `root`
* Deploy the ExternalSecrets operator in the namespace `es`
* Create a `ClusterSecretStore` called `vault-backend`

Once you have this, you can create `ExternalSecret` resources that
read secrets from the vault. See `extsecret-example.yaml` for an example.

## Prerequisites

Hard requirements:

* [helm](https://helm.sh) (`brew install helm`)
* [helmfile](https://helmfile.readthedocs.io/) (`brew install helmfile`)

Soft requirements:

* The [helm diff plugin](https://github.com/databus23/helm-diff) 
  (`helm plugin install https://github.com/databus23/helm-diff`). This is necessary
  if you plan to use `helmfile apply` and `helmfile diff`
* [k3d](k3d.io/) to be able to spawn a local Kubernetes cluster


## Installation

First, start a k3d cluster: 
```bash
k3d cluster create test --api-port 6550 -p "80:80@loadbalancer"
```

Install everything by running the following at the root of this repository:
```bash
helmfile sync
```

Done! Now, test the operator by running: 
```bash
kubectl apply -f extsecret-example.yaml
```

## Accessing the vault

The setting above automatically creates the `secret/foo` for you. To access the vault
interface and add more secrets, create a port forward to access the vault: 
```bash
kubectl port-forward -n vault vault-0 8200
```

You can now go to http://localhost:8200 and log in with the default token `root`.

To access the vault using the command line (and assuming the port-forwarding is on):
```bash
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root

vault kv get secret/foo
```

You can also set secrets programmatically using `kubectl exec`:
```bash
kubectl exec vault-0 -n vault -- vault kv put secret/foo app-secret-key=123
```