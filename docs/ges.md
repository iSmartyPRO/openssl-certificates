# GES Certificate Authority

## Create Intermediate Certificate

### Generate Intermediate Certificate Key

```
openssl genrsa -aes256 -out ./intermediate/private/ges.key.pem 4096

```

### Prepare CSR File

Edit file ./intermediate/openssl-cnf/ges.cnf, especially block [ req_distinguished_name ]

```
openssl req -config ./intermediate/openssl-cnf/ges.cnf -new -sha256 -key ./intermediate/private/ges.key.pem -out ./intermediate/csr/ges.csr.pem
```

### Generate Intermediate Certificate
```
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca -days 1825 -notext -md sha256 -in ./intermediate/csr/ges.csr.pem -out ./intermediate/certs/ges.cert.pem
```