# New external secrets operator

## Local setup

1. Ensure you have [k3d](k3d.io/), `helm` and `helmfile` installed
2. Start a k3d cluster: 
   ```bash
   k3d cluster create test --api-port 6550 -p "80:80@loadbalancer"
   ```
3. Install vault and external-secrets operator: `helmfile sync`
4. test the operator by running: `kubectl apply -f extsecret-example.yaml`

## Accessing the vault

The setting above automatically creates the `secret/foo` for you. To access the vault
interface and add more secrets, create a port-forward to access the vault: 
```bash
kubectl port-forward -n vault vault-0 8200
```

You can now go to http://localhost:8200 and login with the default token `root`.

To access the vault using the commandline (and assuming the port-forwarding is on):
```bash
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root

vault kv get secret/foo
```