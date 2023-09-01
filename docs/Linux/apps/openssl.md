title: OpenSSL


# **OpenSSL**


### **Certificates**

Print info about a certificate in...

* p7b PEM: `openssl pkcs7 -print_certs -inform PEM -in ~/CertificationAuthority.crt`
* p7b DER: `openssl pkcs7 -print_certs -inform DER -in ~/CertificationAuthority.crt`
* x509 PEM: `openssl x509 -inform pem -text -noout -in ~/CertificationAuthority.crt`
* x509 DER: `openssl x509 -inform der -text -noout -in ~/CertificationAuthority.crt`

### **Create bitcoin keys**

See this [stackexchange](https://bitcoin.stackexchange.com/questions/59644/how-do-these-openssl-commands-create-a-bitcoin-private-key-from-a-ecdsa-keypair).
