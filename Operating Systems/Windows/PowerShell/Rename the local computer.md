---
created: 2024-02-22T09:08
updated: 2024-02-22T09:09
tags:
  - Windows
---
https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-computer?view=powershell-7.4

- Rename the local computer :

This command renames the local computer to Server044 and then restarts it to make the change effective.

```
Rename-Computer -NewName "Server044" -DomainCredential Domain01\Admin01 -Restart
```

- Rename a remote computer :

This command renames the Srv01 computer to Server001. The computer is not restarted.

The DomainCredential parameter specifies the credentials of a user who has permission to rename computers in the domain.

The Force parameter suppresses the confirmation prompt.

```
Rename-Computer -ComputerName "Srv01" -NewName "Server001" -DomainCredential Domain01\Admin01 -Force
```