# ASA Certififcate Authority

## Create Intermediate Certificate

### Generate Intermediate Certificate Key

```
openssl genrsa -aes256 -out ./intermediate/private/asa.key.pem 4096

```

### Prepare CSR File

Edit file ./intermediate/openssl-cnf/asa.cnf, especially block [ req_distinguished_name ]

```
openssl req -config ./intermediate/openssl-cnf/asa.cnf -new -sha256 -key ./intermediate/private/asa.key.pem -out ./intermediate/csr/asa.csr.pem
```

### Generate Intermediate Certificate
```
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca -days 1825 -notext -md sha256 -in ./intermediate/csr/asa.csr.pem -out ./intermediate/certs/asa.cert.pem
```


## Create domain certificate

### demo.tc-professional.com

#### Generate domain key
```
openssl genrsa -aes256 -out ./asa/domains/private/demo.tc-professional.com.key.pem 2048
```

#### Prepare CSR File for Certificate

```
openssl req -config ./domains/asa/openssl-cnf/demo.tc-professional.com.cnf -key ./domains/asa/private/demo.tc-professional.com.key.pem -new -sha256 -out ./domains/asa/csr/demo.tc-professional.com.csr.pem -addext "subjectAltName = DNS:demo.tc-professional.com"
```

#### Sign and generate domain certificate

```
openssl ca -config ./domains/asa/openssl-cnf/demo.tc-professional.com.cnf -extensions server_cert -days 2000 -notext -md sha256 -in ./domains/asa/csr/demo.tc-professional.com.csr.pem -out ./domains/asa/certs/demo.tc-professional.com.cert.pem
```