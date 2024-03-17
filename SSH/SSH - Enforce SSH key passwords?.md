---
created: 2024-02-28T16:53
updated: 2024-02-28T17:02
tags:
  - SSH
  - openSSH
  - Security
  - Cybersecurity
related link: https://serverfault.com/a/82655
---
The passphrase that can be set on the private key is **unrelated to the SSH server** or the connection to it. Setting a passphrase to the private key is merely a security measure the key owner may take in order to prevent access to his remote shell by a third party in case the private key gets stolen.

Unfortunately, you cannot force users to secure their private keys with passphrases. Sometimes, unprotected private keys are required in order to 
automate access to the remote SSH server.

___

You can't. Once the user has the key data in their possession, you can't stop them from removing the passphrase. You'll need to look for other ways of doing your authentication.

___

One way to ensure that a passphrase is used is a formal policy with a penalty for keys found unencrypted. But it is not hard or inconvenient to use ssh-keys properly.
