# Vault charts

There are 2 charts in this repo:

* Vault
* Vault-ui

[Vault](https://www.vaultproject.io) secures, stores, and tightly controls access to tokens, passwords, certificates, API keys, and other secrets in modern computing. Vault handles leasing, key revocation, key rolling, and auditing. Through a unified API, users can access an encrypted Key/Value store and network encryption-as-a-service, or generate AWS IAM/STS credentials, SQL/NoSQL databases, X.509 certificates, SSH credentials, and more.

## Running the helm chart

The easiest way of running vault is using the `dev` mode (please, do not use it in production).

```
helm install charts/vault"
```

If you want to use the production mode you have to update the `config.json` key in the secret. Currently it is set up to use AWS S3 as the backend:

```
storage "s3" {
  access_key = "xxx"
  secret_key = "xxx"
  bucket     = "vault-bucket"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}
```


### Seal/Unseal process

Unsealing makes the process of automating a Vault install difficult. Automated tools can easily install, configure, and start Vault, but unsealing it is a very manual process.

This complicates the use of Vault inside of Kubernetes, because if the pod dies, Kubernetes will start the pod again but it will be sealed.

When Vault starts it has to be initialised:

```
vault init

Unseal Key 1: YE8GePrAGb2dH0/O/Tzz2JxNuK9PWoEY8wHJ6v
Unseal Key 2: mNlbiZXxvaKwIKMAuAjAnWPeesWJS81i1ACvQ7
Unseal Key 3: tkDd5bCpFaUuF4S9iiBf2CrM//fT+aHYBrcwVc
Unseal Key 4: 1iUerpWSW6jdj3EtXiY9W10S0ngYdonQ8do8KQ
Unseal Key 5: fNNonAC4Fe6XScR/oHlhzPFW9rQeoEj1lL/Ktk
Initial Root Token: cd6edb9a-11f51-b14d-f7568f6e2661

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your vault will remain permanently sealed.
```

Once the vault has been initialised, it has to be unsealed:

```
vault unseal YE8GePrAGb2dH0/VxLt3ZO/Tzz2JxNuK9PWoEY8wHJ6v
vault unseal mNlbiZXxvaKwIKMAuzUymJxAjAnWPeesWJS81i1ACvQ7
vault unseal tkDd5bCpFaUuF4SLCjLzW9iiBf2CrM//fT+aHYBrcwVc
```

## Running Vault-ui

Once the Vault has been unsealed, you can install a UI. This repo contains a chart with the [vault-ui](https://github.com/djenriquez/vault-ui) project by [DJ Eniquez](https://github.com/djenriquez).

To run this chart you need 2 settings: 

* VAULT_URL_DEFAULT: http://vault-service-name:8200
* VAULT_AUTH_DEFAULT: by default is token, but you can use any of the 4 options provided.


```
helm install charts/vault-ui --set vault.url=http://MY_RELEASE-vault:8200"
```
