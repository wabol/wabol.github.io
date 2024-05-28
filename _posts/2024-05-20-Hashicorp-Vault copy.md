---
title: Hashicorp Vault
date: 2024-05-20 11:31:00 -0700
categories: [Vault]
tags: 
Pin:
---

### What is the Hashicorp Vault key feature

HashiCorp Vault is an open-source tool designed for securely managing secrets and protecting sensitive data. Here are some key features of Vault:
1. Secret Storage: Vault provides a secure storage mechanism to store and access secrets. Secrets can be anything from database credentials to API keys.
2. Dynamic Secrets: Vault can generate secrets dynamically, such as database credentials or cloud access tokens, ensuring that these secrets have a limited lifespan and reducing the risk of misuse.
3. Data Encryption: Vault encrypts data both in transit and at rest, ensuring that the stored secrets are secure from unauthorized access.
4. Access Control: Vault has a robust access control system that uses policies to define who can access which secrets. This ensures that only authorized applications and users can access sensitive data.
5. Audit Logging: Vault logs all access requests and actions, providing a detailed audit trail that can be used for compliance and security monitoring.
6. Identity-Based Access: Vault integrates with various identity providers to enforce access controls based on user identity. This includes integration with LDAP, Active Directory, and cloud identity services.
7. Secret Engines: Vault supports various secret engines, which are responsible for managing specific types of secrets. Examples include the KV (key-value) engine, the database engine, and the PKI engine.
8. Vault Agent: A client-side daemon that automates authentication and secret retrieval, reducing the complexity of integrating applications with Vault.

### How to use Vault

1. #### Install Vault
   Please acces the official website to install. Below is the link: https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install

2. #### Start Vault
   After installing Vault, you can start it by running the following command:
   ```shell
   vault server -config=/path/to/vault_config.hcl
   ```
   The config file should be in the format of HCL (Hashicorp Configuration Language). Here is an example of the config file:
   ```shell
   ui            = true

   storage "file" {
   path = "/path/to/vault/data"
   }

   listener "tcp" {
   address       = "127.0.0.1:8200"
   tls_disable = 1
   }
   ```
   This config file will start Vault in a product mode with a file storage backend and a TCP listener on port 8200. If you want to use development mode, you can start with `vault server -dev`
   If you want to use a different storage backend or listener, please refer to the Vault documentation. https://developer.hashicorp.com/vault/docs/configuration/storage

3. #### Initialize Vault
   After starting Vault, you need to initialize it by running the following command:
   ```shell
   vault operator init
   ```
   This command will generate a root token and a unseal key. The root token is used to authenticate with Vault and the unseal key is used to unseal the Vault.
   You can also specify the number of shares and threshold for the unseal key.

   Another way to initialize Vault is open the browser and access http://127.0.0.1:8200 to initialize it.

4. #### Unseal Vault and login
   After initializing Vault, you need to unseal it by running the following command:
   ```shell
   vault operator unseal
   ```
   Use root login
   ```shell
   vault login
   ```
5. #### Enable KV engine and create, read, update, delete a secret
   To enable the KV engine, run the following command:
   ```shell
   vault secrets enable -path="kv-v1" -description="My first engine" kv
   ```
   List enabled secrets engines:
   ```shell
   vault secrets list
   ```
   Create the 1st secret:
   ```shell
   vault kv put kv-v1/hello foo=bar dummy=demo
   ```
   Read the secret:
   ```shell
   vault kv get kv-v1/hello
   ```
   Update the secret:
   ```shell
   vault kv put kv-v1/hello foo=bar2 dummy=demo2
   ```
   Read the different version of the secret:
   ```shell
   vault kv get -version=1 kv-v1/hello
   ```
   Delete the secret:
   ```shell
   vault kv delete kv-v1/hello
   ```
6. #### Enable transit engine and encrypt, decrypt data
   To enable the transit engine, run the following command:
   ```shell
   vault secrets enable -path="transit" -description="My transit engine" transit
   ```
   Create a key:
   ```shell
   vault write -f transit/keys/my-key
   ```
   Encrypt data (base64 encoded):
   ```shell
   vault write transit/encrypt/my-key plaintext=$(base64 <<< "mysecrest data")
   ```
   Decrypt data:
   ```shell
   vault write transit/decrypt/my-key ciphertext="vault:v1:OS9cKsPwnaI7Vaq8J4a+/iP6qF90Atgezveqz9tNpss+6ubU/aSQSWru7A=="
   # The result is "bXlzZWNyZXN0IGRhdGEK"
   ```
   Base64 decode the decrypted data:
   ```shell
   echo "bXlzZWNyZXN0IGRhdGEK" | base64 --decode
   # The result is "mysecrest data"
   ```
7. #### Enable database engine and connect to a database
   Enable the database engine:
   ```shell
   vault secrets enable database
   ```
   Configure the database engine:
   ```shell
   vault write database/config/my-mysql-database \
    plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
    allowed_roles="my-role" \
    username="root" \
    password="rootpassword"
   ```
    Create a role for the database:
   ```shell
   vault write database/roles/my-role \
    db_name=my-mysql-database \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT ALL PRIVILEGES ON *.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="24h"
   ```
   Create a policy for the database:
   ```shell
   vault policy write db-policy.hcl - <<EOF
   path "database/creds/my-mysql-database" {
       capabilities = ["read"]
   }
   EOF
   ```
   Apply the policy to a user:
   ```shell
   vault policy write db-policy db-policy.hcl
   ```
   Create a AppRole for the database:
   ```shell
   vault write auth/approle/role/my-app-role token_policy="db-policy"
   ```
   Get a token for the AppRole:
   ```shell
   vault read auth/approle/role/my-app-role/role-id
   vault write -f auth/approle/role/my-app-role/secret-id
   ```
   Use the token to connect to the database:
   ```shell
   # Vault address and Approle config
   VAULT_ADDR="http://127.0.0.1:8200"
   ROLE_ID="<your_role_id>"
   SECRET_ID="<your_secret_id>"

   # Get a token for the AppRole
   VAULT_TOKEN=$(curl -s --request POST --data "{\"role_id\": \"${ROLE_ID}\", \"secret_id\": \"${SECRET_ID}\"}" ${VAULT_ADDR}/v1/auth/approle/login | jq -r '.auth.client_token')

   # Get dynamic credentials from Vault
   DB_CREDENTIALS=$(curl -s --header "X-Vault-Token: ${VAULT_TOKEN}" ${VAULT_ADDR}/v1/database/creds/my-role)

   DB_USERNAME=$(echo $DB_CREDENTIALS | jq -r '.data.username')
   DB_PASSWORD=$(echo $DB_CREDENTIALS | jq -r '.data.password')

   # Print the credentials
   echo "Username: $DB_USERNAME"
   echo "Password: $DB_PASSWORD"

   # Connect to the database using the credentials
   mysql -u $DB_USERNAME -p$DB_PASSWORD -h 127.0.0.1 my-mysql-database -p 3306
   ```