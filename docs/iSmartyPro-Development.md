# iSmartyPro Development Certificate Authority

## Create Intermediate Certificate

### Generate Intermediate Certificate Key

```
openssl genrsa -aes256 -out ./intermediate/private/iSmartyPro-Development.key.pem 4096

```

### Prepare CSR File

Edit file ./intermediate/openssl-cnf/iSmartyPro-Development.cnf, especially block [ req_distinguished_name ]

```
openssl req -config ./intermediate/openssl-cnf/iSmartyPro-Development.cnf -new -sha256 -key ./intermediate/private/iSmartyPro-Development.key.pem -out ./intermediate/csr/iSmartyPro-Development.csr.pem
```

### Generate Intermediate Certificate
```
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca -days 1825 -notext -md sha256 -in ./intermediate/csr/iSmartyPro-Development.csr.pem -out ./intermediate/certs/iSmartyPro-Development.cert.pem
```