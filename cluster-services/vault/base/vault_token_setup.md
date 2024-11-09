### Goal
Vault tokens by default have an expiration time. After this token expires, external-secrets can't fetch latest secrets from
vault server.
So we need to create a periodic token. Then we will need a job that will renew this periodic token continously.

Follow the steps below.

#### 1. Create the periodic token

First, set the `root_token` environment variable (Refer Bitwarden)
```
curl \
    --header "X-Vault-Token: $root_token" \
    --request POST \
    --data @create_config.json \
    https://vaultui-central.saifty.coacapp.de/v1/auth/token/create
```



*TODO* 
Currently we are attaching `default` and `heraeus` policy to this token. But ideally, we should create a custom policy that 
only allow specific endpoints like : 
- token renewal : `v1/auth/token/renew-self`
- accessing the heraeus-dev vault secrets at : `https://vaultui-central.saifty.coacapp.de/ui/vault/secrets/heraeus-dev`


#### 2. Create the secret using this token : 
```
apiVersion: v1
kind: Secret
metadata:
  name: vault-token
  namespace: external-secrets
data:
  token: <PASTE_VAULT_TOKEN_HERE>
```

#### 2. Also Save the token it in vault server heraeus secret engine.
Ensure the secret `periodic-token-vault` has the periodic-token created above.
- Refer to vault-token-renew-job.yaml
  This job will keep on renewing the periodic-token.
