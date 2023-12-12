# Important Links #links/vault/associate
- [GitHub](https://github.com/btkrausen/hashicorp)
- [YouTube - Vault Certification Series](https://www.youtube.com/c/NedintheCloud)
- [Preparation Guide](https://www.leanpub.com/vault-certified)
- [Hands-On Lab - Kodekloud](https://kodekloud.com/courses/lab-hashicorp-certified-vault-associate-certification/)
	- Coupon code - M2VCSS82
# Vault Installation
1. Install Vault
2. Create config file
3. Initialize Vault
4. Unseal Vault
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault
```
## Run Vault server in dev mode #commands/vault/dev-mode
```sh
vault server -dev

# In a new shell
export VAULT_ADDR='http://127.0.0.1:8200'
vault status
vault secrets list
vault kv put secret/learn/vaultpath planet=middleearth
vault kv get secret/learn/vaultpath
```
## Run Vault in a cluster configured with integrated storage (Raft)

```sh
vault operator raft list-peers
vault operator raft join http://node-1:8200

vault operator unseal
```
## Vault commands - Initialize #commands/vault/initialize

## Vault commands - Initialize #commands/vault/seal-unseal
