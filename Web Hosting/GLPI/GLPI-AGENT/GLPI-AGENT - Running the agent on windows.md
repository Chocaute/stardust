---
created: 2024-02-22T14:56
updated: 2024-02-22T14:58
tags:
  - GLPI
  - WebHosting
  - Windows
related link: https://glpi-agent.readthedocs.io/en/latest/usage.html
---
### HTTP interface

If the agent is running as a service or a daemon, its web interface should be accessible at `http://hostname:62354`.

If the machine connecting to this interface is trusted (see [httpd-trust configuration directive](https://glpi-agent.readthedocs.io/en/latest/configuration.html#httpd-trust)), a link will be available to force immediate execution.

### Command line

> [!NOTE]
> Don't use **PowerShell** as commandline interpreter: it changes some environment context the agent doesn't handle well.
> 

Open a command interpreter windows (`cmd.exe`), with administrator privileges (_right click_, _Run as Administrator_).

Go to `C:\Program files\GLPI-Agent` (adapt path depending on your configuration) folder to run it:
```cmd
cd "C:\Program files\GLPI-Agent"
glpi-agent
```