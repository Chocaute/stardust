---
created: 2024-06-04T15:59
updated: 2024-10-11T13:52
tags:
  - Apache2
  - Certificates
  - WebHosting
---

> [!INFO] Requirements
> You'll need a Certificate Authority. 
> To create your own, follow these steps: [[Quick Certificate Authority Setup]]

### Set Up the server SSL Certificate

[](Quick%20Certificate%20Authority%20Setup.md)y%20setup.md)y%20setup.md)shell
openssl genrsa -out httpd.key 2048
```

 2. **Create a Certificate Signing Request (CSR)**

Create a configuration file `httpd_openssl.cnf`:
```shell
[ req ] 
default_bits       = 2048 
distinguished_name = req_distinguished_name 
req_extensions     = req_ext 
prompt             = no  

[ req_distinguished_name ] 
C  = AU 
ST = Some-State 
L  = Some-City 
O  = Your-Organization 
CN = 192.168.3.2  

[ req_ext ] 
subjectAltName = @alt_names  

[ alt_names ] 
IP.1 = 192.168.3.2 
DNS.1 = your.hostname
```

3. **Generate the CSR using this configuration file**

```shell
openssl req -new -key httpd.key -out httpd.csr -config httpd_openssl.cnf
```

#### CA-side

4. **Sign the CSR with the CA to create the server Certificate**

```shell
openssl x509 -req -in httpd.csr -CA ~/myCA/myCA.crt -CAkey ~/myCA/myCA.key \
-CAcreateserial -out httpd.crt -days 365 -sha256 \
-extfile httpd_openssl.cnf -extensions req_ext
```

This signs the GLPI server CSR with the CA’s private key and certificate to create the GLPI server certificate (`glpi.crt`).

5. **Create a Full Chain Certificate**

Concatenate the GLPI server certificate and the CA certificate:

```bash
cat glpi.crt ~/myCA/myCA.crt > glpi_fullchain.crt
```

### Configure Apache

#### Server-side

1. **Copy Certificates to Appropriate Locations**

```bash
sudo cp httpd.key /etc/ssl/private/
sudo cp httpd_fullchain.crt /etc/ssl/certs/
```

2. **Update Apache Configuration**

Edit the Apache virtual host configuration to use the new certificates:

```apacheconf
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/glpi_fullchain.crt
    SSLCertificateKeyFile /etc/ssl/private/glpi.key
[...]
```


> [!Tip] For web-browsing
> If you import the CA certificate in your favorite browser (or in your OS), you will not get a "self-signed" alert when connecting to your website if it has a self-signed certificate.
