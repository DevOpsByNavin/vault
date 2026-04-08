## Path based workflow

Everything inside a vault is path based.  

It doesn't necessarily represent the actual folder path but logically mapped to the actual storage.

That means, you store secrets to specific path.  

If you create policy then you will use `.hcl` (HashiCorp Corporation Language) format to define what permission are allowed on what path. Then you gonna attach that policy to an identity.

Sample policy:

```hcl
path "secret/data/myapp/*" {
    capabilities = ["read"]
}

path "secret/metadata/myapp/*" {
    capabilities = ["list"]
}
```

## Secret Engines

They are like a plugin that handles various category of secrets. Common types includes:

1. KeyValue pair: 
    - kv-v1: Basic key value
    - kv-v2: allow versioning, default to soft delete

2. PKI
3. Transit
4. Database and many more

To use secret engine you have to first mount it to vaults path.

## Authentication Method

Vault supports various way to identify an user. By default you login with your root token, but you are recommended to create a userpass login method with enough permission attached to carry out day-day operation.

1. Token
2. Usepass
3. approle
4. github login and many more

## Vaults Cryptography

```
================================================================================
                                VAULT RUNTIME (RAM)
--------------------------------------------------------------------------------
         [ Human Input ]                  [ Cryptographic Barrier ]
                                       <--- (RAM/Barrier Divider) --->
  +-----------------------+                    |
  | Unseal Key 1 (Partial)|---.                |
  +-----------------------+   |                |
                              | [RECONSTRUCT]  |  +-------------------------+
  +-----------------------+   +----------------|->|        MASTER KEY       |
  | Unseal Key 2 (Partial)|---| (in memory only)|  | (Stored Encrypted by LB)|*
  +-----------------------+   |                |  +-------------------------+
                              |                |             |
  +-----------------------+---'                |             | [DECRYPTS]
  | Unseal Key N (Partial)|                    |             V
  +-----------------------+                    |  +-------------------------+
                                               |  |       KEYRING           |
                                               |  | +---------------------+ |
         (Threshold Met)                       |  | | [ACTIVE KEY] <---. | |
                                               |  | +---------------------+ |
                                               |  | | (Archived Key 1) | | |
===============================================|==|=========================|===
        PHYSICAL STORAGE (./data directory)    |  |-------------------------|
===============================================|==|=========================|===
                                               |  |                         |
                                               |  +-------------------------+
                                               |             |
                                               |             | [WRAPS DEK]
                                               |             V
  +------------------------+                   |  +-------------------------+
  |    USER'S DATA         |                   |  | DATA ENCRYPTION KEY (DEK)|
  | (e.g., password=123)   |---[ENCRYPT]-------|->| (Generated Unique per   |
  +------------------------+      | (using DEK)|  | entry/request)         |
                                  |            |  +-------------------------+
                                  |            |             |
                                  |            |             | [WRAPS DATA]
                                  V            |             V
  +------------------------------------------------------------------------+
  |                   ENCRYPTED BLOB (SENT TO BACKEND)                      |
  |  | Encrypted User Data     |        | Encrypted Data Encryption    |   |
  |  |                         |        | Key (DEK)                    |   |
  |  +-------------------------+        +------------------------------+   |
  +------------------------------------------------------------------------+
                               |
                               +---> stored in data/ folder
                                     as a standard (but unreadable) file.
================================================================================
*Note on standard setup: Vault stores the Master Key encrypted on the backend.
The only way to decrypt it is by providing the correct unseal keys.
In cloud setups (AWS KMS, etc.), the master key might be stored in the cloud HSM.
```

## API Usages

Base URL: `https://vault.example.com:8200/v1/<path>`  
Headers: `X-Vault-Token: <token>`

Simple curl request: 

```bash
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/secret/data/myapp/config | jq .
```