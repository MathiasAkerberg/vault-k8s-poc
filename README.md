# POC Vault-K8S

This is a simple demo of how Vault secrets could be [injected into Kubernetes Pods via a Sidecar](https://www.hashicorp.com/blog/injecting-vault-secrets-into-kubernetes-pods-via-a-sidecar/) in a .NET Core project.

```
kubectl create namespace vault-poc

helm repo add hashicorp https://helm.releases.hashicorp.com
helm install -n vault-poc \
       --set='server.dev.enabled=true' \
       --version=0.6.0 \
       vault hashicorp/vault 
```

```
kubectl exec -ti vault-0 /bin/sh

cat <<EOF > /home/vault/webapp-policy.hcl
path "secret*" {
  capabilities = ["read"]
}
EOF

vault policy write webapp /home/vault/webapp-policy.hcl
```

```
vault auth enable kubernetes

vault write auth/kubernetes/config \
   token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
   kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
   kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

vault write auth/kubernetes/role/webapp \
   bound_service_account_names=webapp \
   bound_service_account_namespaces=vault-poc \
   policies=webapp \
   ttl=1h
```

vault kv put --address=http://127.0.0.1:59818 secret/webapp/appsettings "@appsettings.Production.json"
