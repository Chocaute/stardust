---
created: 2024-02-28T17:01
updated: 2024-02-28T17:02
tags:
  - SSH
  - openSSH
  - Cybersecurity
related link: https://serverfault.com/a/82655
---
> One good habit I **highly recommend** for such cases is to advise the users to **hash** the _known_hosts_ file (stored at _~/.ssh/known_hosts_), which keeps information about the remote hosts the user connects to.
 
Using the following command:
```shell
ssh-keygen -H -f ~/.ssh/known_hosts
```

This way, even if a third party gained access to an unprotected private key, it would be extremely difficult to find out for which remote hosts this key is valid. Of course, clearing the shell history is mandatory for this technique to be of any value.