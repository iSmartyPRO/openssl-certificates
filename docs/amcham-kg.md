# AmCham Kyrgyzstan Certificate Authority

## Create Intermediate Certificate

### Generate Intermediate Certificate Key

```
openssl genrsa -aes256 -out ./intermediate/private/amcham-kg.key.pem 4096

```

### Prepare CSR File

Edit file ./intermediate/openssl-cnf/amcham-kg.cnf, especially block [ req_distinguished_name ]

```
openssl req -config ./intermediate/openssl-cnf/amcham-kg.cnf -new -sha256 -key ./intermediate/private/amcham-kg.key.pem -out ./intermediate/csr/amcham-kg.csr.pem
```

### Generate Intermediate Certificate
```
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca -days 1825 -notext -md sha256 -in ./intermediate/csr/amcham-kg.csr.pem -out ./intermediate/certs/amcham-kg.cert.pem
```