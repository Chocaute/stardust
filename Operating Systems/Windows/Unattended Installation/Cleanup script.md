---
created: 2023-12-06T10:53
updated: 2023-12-22T09:24
---
#Windows #Automation #Deployment 

https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-a-custom-script-to-windows-setup?view=windows-10#use-unattend-to-run-scripts

## Use Unattend to run scripts

Create an Unattend.xml file with one of these settings to run during the Windows Setup process. This can be used with OEM product keys.

To run services or commands that can start at the same time, use RunAsynchronousCommands.

Some of these settings run in the user context, others run in the system context depending on the configuration pass.

- Add [Microsoft-Windows-Setup\RunAsynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-setup-runasynchronous-runasynchronouscommand) or [RunSynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-setup-runsynchronous-runsynchronouscommand) to run a script as Windows Setup starts. This can be helpful for setting hard disk partitions.
    
- Add [Microsoft-Windows-Deployment\RunAsynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-deployment-runasynchronous-runasynchronouscommand) or [RunSynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-deployment-runsynchronous-runsynchronouscommand) to the **auditUser** configuration pass to run a script that runs when the PC enters audit mode. This can be helpful for tasks like automated app installation or testing.
    
- Add [Microsoft-Windows-Shell-Setup\LogonCommands\AsynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-logoncommands) or [FirstLogonCommands\SynchronousCommand](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-firstlogoncommands) to run after the Out of Box Experience (OOBE) but before the user sees the desktop. This can be especially useful to set up language-specific apps or content after the user has already selected their language.
    
    Use these scripts sparingly because long scripts can prevent the user from reaching the Start screen quickly. For retail versions of Windows, additional restrictions apply to these scripts. For info, see the Licensing and Policy guidance on the [OEM Partner Center](https://go.microsoft.com/fwlink/?LinkId=131358).


## Run a script after setup is complete (SetupComplete.cmd)

### Order of operations

1. After Windows is installed but before the logon screen appears, Windows Setup searches for the **SetupComplete.cmd** file in the ```%WINDIR%\Setup\Scripts\``` directory.

3. If a **SetupComplete.cmd** file is found, Windows Setup runs the script. Windows Setup logs the action in the ```C:\Windows\Panther\UnattendGC\Setupact.log``` file.

4. On the deployment base image computer open Notepad and enter in the following lines:
```
del /Q /F c:\windows\system32\sysprep\unattend.xml  
del /Q /F c:\windows\panther\unattend.xml
```

These lines of code will delete the unattend.xml file from the computer once the Windows Setup is finished with them (this file is copied into the panther directory during setup hence the two lines)

6. Save this file to the desktop called SetupComplete.cmd

7. Now create a folder called ```Scripts``` in this directory: ```C:\Windows\Setup\``` and drag this file into it (you may be prompted for Administrator authority).
