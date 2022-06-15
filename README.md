# Generate self signed certs TLS certificates

Commands to generate self signed certs, bundle them in keystore and truststores. 

### Create your CA

1. Create a private key for your CA.
```
openssl genrsa -aes256 -out ca-private-key.pem 4096
```
2. Create your CA public cert with key from step 1.
```
openssl req -new -x509 -sha256 -days 365 -key ca-private-key.pem -out ca-public-cert.pem
```

### Create your server cert

3. Create a server private key.
```
openssl genrsa -out server-private-key.pem 4096
```

4. Create a signing request.
```
openssl req -new -sha256 -subj "/CN=jhugh02-vm2" -key server-private-key.pem -out cert_request.csr
```

5. Create a config file to provide subjectAltName's, Add all/any IPs or DNS you need.
```
echo "subjectAltName=DNS:reproduce1,DNS:localhost,IP:172.31.44.23" >> extfile.cnf
```
6. Create your new signed certificate from the csr
```
openssl x509 -req -sha256 -days 365 -in cert_request.csr -CA ca-public-cert.pem -CAkey ca-private-key.pem -out signed-cert.pem -extfile extfile.cnf -CAcreateserial
```
### Package keys and certs.

8. Chain your CA and server certificates.
```
cat signed-cert.pem > jhugh02-vm2_chain.pem
cat ca-public-cert.pem >> jhugh02-vm2_chain.pem
```
8. Bundle your server key and chained certificate together
```
openssl pkcs12 -export -in reproduce1_chain.pem -inkey server-private-key.pem -name reproduce1 -out reproduce1.p12
```
9. Package your cert and key bundle into a java keystore for LDM.
keytool -importkeystore -deststorepass wandisco -destkeystore keystore.jks -srckeystore jhugh02-vm2.p12 -srcstoretype PKCS12

10. Package your CA cert into a java truststore for LDM.
```
keytool -import -alias CA -file ca-public-cert.pem -keystore truststore.jks -storepass wandisco
```

