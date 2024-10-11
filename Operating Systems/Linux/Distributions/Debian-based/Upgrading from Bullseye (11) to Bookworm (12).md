---
created: 2023-12-21T16:36
updated: 2024-03-12T15:45
---
#Linux #Debian 

https://www.cyberciti.biz/faq/update-upgrade-debian-11-to-debian-12-bookworm/

The procedure is as follows:

1. Backup the system.
   
2. Update existing packages and reboot the Debian 11 system.
   
3. Edit the file ``/etc/apt/sources.list`` using a text editor and replace each instance of **bullseye** with **bookworm**. Next find the update line, replace keyword **bullseye-updates** with **bookworm-updates**. Finally, search the security line, replace keyword **bullseye-security** with **bookworm-security**.
   
4. Update the packages index on Debian Linux, run:  
```shell
sudo apt update
```

5. Prepare for the operating system minimal system upgrade, run:  
```shell
sudo apt upgrade --without-new-pkgs
```

6. Finally, update Debian 11 to Debian 12 Bookworm by running:  
```shell
sudo apt full-upgrade
```

7. Reboot the Linux system so that you can boot into Debian 12 Bookworm
8. Verify that everything is working correctly.

Let us see all commands step by step to **upgrade Debian 11 Bullseye to Debian 12 Bookworm safely** running in the cloud or bare metal environment.