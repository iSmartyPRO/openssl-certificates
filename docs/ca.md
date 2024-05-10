# Generate iSmartyPro Global Root Certificate Authority

## Create Private Key
```
openssl genrsa -aes256 -out ./ca/private/ca.key.pem 4096
```

## Create Root Certificate
```
openssl req -config ./ca/openssl.cnf -new -x509 -days 10950 -sha256 -extensions v3_ca -key ./ca/private/ca.key.pem -out ./ca/certs/ca.cert.pem
```