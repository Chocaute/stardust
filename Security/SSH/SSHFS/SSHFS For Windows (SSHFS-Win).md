---
created: 2024-05-17T19:06
updated: 2024-05-17T19:07
tags:
  - Windows
  - SSHFS
  - SSH
related link: https://github.com/winfsp/sshfs-win
---
# About

SSHFS-Win is a minimal port of [SSHFS](https://github.com/libfuse/sshfs) to Windows. Under the hood it uses [Cygwin](https://cygwin.com) for the POSIX environment and [WinFsp](https://github.com/billziss-gh/winfsp) for the FUSE functionality.

## Installation

- Install the latest version of [WinFsp](https://github.com/billziss-gh/winfsp/releases/latest).
- Install the latest version of [SSHFS-Win](https://github.com/billziss-gh/sshfs-win/releases). Choose the x64 or x86 installer according to your computer's architecture.

Both can also be easily installed with [WinGet](https://github.com/microsoft/winget-cli):

```shell
winget install -h -e --id "WinFsp.WinFsp" ; winget install -h -e --id "SSHFS-Win.SSHFS-Win"
```

## Basic Usage

Once you have installed WinFsp and SSHFS-Win you can map a network drive to a directory on an SSHFS host using Windows Explorer or the `net use` command.

### Windows Explorer

In Windows Explorer select This PC > Map Network Drive and enter the desired drive letter and SSHFS path using the following UNC syntax:

```
\\sshfs\REMUSER@HOST[\PATH]
```

The first time you map a particular SSHFS path you will be prompted for the SSHFS username and password. You may choose to save these credentials with the Windows Credential Manager in which case you will not be prompted again.

In order to unmap the drive, right-click on the drive icon in Windows Explorer and select Disconnect.

[![](https://github.com/winfsp/sshfs-win/raw/master/cap.gif)](https://github.com/winfsp/sshfs-win/blob/master/cap.gif)

### Command Line

You can map a network drive from the command line using the `net use` command:

```
> net use X: \\sshfs\billziss@mac2018.local
The password is invalid for \\sshfs\billziss@mac2018.local.

Enter the user name for 'sshfs': billziss
Enter the password for sshfs:
The command completed successfully.
```

You can list your `net use` drives:

```
$ net use
New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
             X:        \\sshfs\billziss@mac2018.local
                                                WinFsp.Np
The command completed successfully.
```

Finally you can unmap the drive as follows:

```
$ net use X: /delete
X: was deleted successfully.
```