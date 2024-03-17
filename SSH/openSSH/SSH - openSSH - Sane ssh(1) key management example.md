---
created: 2023-12-21T17:00
updated: 2023-12-21T17:01
---
#SSH #openSSH

https://try.popho.be/ssh-keys.html

It’s common knowledge that you [shouldn’t put all your eggs in the same basket](https://en.wiktionary.org/wiki/don%27t_put_all_your_eggs_in_one_basket), but most of the time on IRC or on reddit (or the Internet at large, really), I see people using one single ssh key for all uses. How would you look at someone using a single key for their car, house, safe, work place, and so on?

```
[...]
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
[...]
```

This causes things like [whoami by Filippo](https://whoami.filippo.io) to work. (NB: it really shouldn’t)

Now, contrary to real lifeTM, `ssh(1)` can automatically find a key without much fiddling with a keyring. [See `man 5 ssh_config`](https://man.openbsd.org/ssh_config#IdentityFile), and the ssh [TOKENS](https://man.openbsd.org/ssh_config#TOKENS).

```
# Magic happens here, and it happens for all hosts
IdentityFile ~/.ssh/keys/%h
# Fallback
IdentityFile ~/.ssh/id_rsa
```

Now, we only need to generate one key per host after we create our directory structure: `umask 077; mkdir -p ~/.ssh/keys`. [ssh-keygen(1)](https://man.openbsd.org/ssh-keygen) does just that.

```
% ssh-keygen -ted25519 -f ~/.ssh/keys/full.host.name
```

```
% ssh whoami.filippo.io
no such identity: /home/moviuro/.ssh/keys/whoami.filippo.io: No such file or directory
no such identity: /home/moviuro/.ssh/id_rsa: No such file or directory

    +---------------------------------------------------------------------+
    |                                                                     |
    |             _o/ Hello!                                              |
    |                                                                     |
    |                                                                     |
    |  Did you know that ssh sends all your public keys to any server     |
    |  it tries to authenticate to? You can see yours echoed below.       |
    |                                                                     |
    |  We tried to use them to lookup your GitHub account,                |
    |  but got no match :(                                                |
    |                                                                     |
    |  -- @FiloSottile (https://twitter.com/FiloSottile)                  |
    |                                                                     |
    |                                                                     |
    |  P.S. The source of this server is at                               |
    |  https://github.com/FiloSottile/whoami.filippo.io                   |
    |                                                                     |
    +---------------------------------------------------------------------+
```

As an added benefit now, if one of your ssh keys ever leaks, there’s only one place to remove it from `~/.ssh/authorized_keys` (where the `login@hostname` comment is still present).