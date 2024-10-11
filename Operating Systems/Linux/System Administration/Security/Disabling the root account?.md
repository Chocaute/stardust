---
created: 2023-12-21T16:28
updated: 2023-12-21T16:34
---
#Linux   #Debian #Security

https://unix.stackexchange.com/questions/383301/should-i-disable-the-root-account-on-my-debian-pc-for-security

There are different reasons to disable root account, for example:

1. Your system is available on a network and you want to protect yourself against brute force attacks, so no one can guess your root account password.
    
2. Developers wants to stop the users from running a command like `su -` to get a full root shell, because it's now a lot easier to do something wrong which causes damage to the system. however they can still use something like `sudo -i`, `sudo -s`, `sudo /bin/some-shell` or even `sudo su -` if they are in sudoers file.
    
    The idea is to force the user to use the `sudo` instead of sharing a single root password between all users and using the `sudo` comes with some advantages, for example:
    
    - It's less likely for you to leave an open shell with complete root access, `sudo` permissions expires after a while.
    - You can define more flexible ruels using `sudoers` file
    - It logs who is doing what, etc.
    - Read [here](https://help.ubuntu.com/community/RootSudo) for more info.

---

To disable, you can remove the password of the account or lock it down, or even do both of them:

1. Remove the root password:

 ```
 sudo passwd -d root
```

2. Lock the account:

```
sudo passwd -l root
```

