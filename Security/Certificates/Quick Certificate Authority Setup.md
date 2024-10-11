---
created: 2024-06-04T15:55
updated: 2024-06-04T15:58
tags:
  - Certificates
  - Linux
---
### Create the CA Private Key and Self-Signed Certificate

```shell
mkdir -p ~/myCA
cd ~/myCA
openssl genrsa -out myCA.key 4096 openssl req -x509 -new -nodes -key myCA.key \
-sha256 -days 3650 -out myCA.crt \
-subj "/C=FR/ST=Loire Atlantique/O=Tom RAUD/CN=MyCA"
```

This creates a directory for the CA, generates a private key (`myCA.key`), and a self-signed certificate (`myCA.crt`).

