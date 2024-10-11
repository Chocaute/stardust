---
created: 2024-06-03T14:02
updated: 2024-06-03T14:21
tags:
  - Linux
  - Ubuntu
  - AppImage
  - AppArmor
related link: https://github.com/arduino/arduino-ide/issues/2429#issuecomment-2099775010
---
### The problem:

```shell
./arduino-ide_2.3.2_Linux_64bit.AppImage 
[49662:0505/163801.040968:FATAL:setuid_sandbox_host.cc(158)] The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /tmp/.mount_arduinl1RTCc/chrome-sandbox is owned by root and has mode 4755.
Trace/breakpoint trap (core dumped)
```

### The (temporary) solution:

The issue is with the AppArmor configuration in Ubuntu 24.04, not the AppImage. The change in the configuration is explained in the release notes of Ubuntu 24.04 (security reasons).

Because this problem is caused by the OS configuration, I'm not use what the Arduino IDE team can do, except for documenting the installation procedure.

You can disable the sandboxing restriction for all program with:

```shell
sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=1
```

or by adding to /etc/sysctl.d/local.conf:

```shell
kernel.apparmor_restrict_unprivileged_userns=0
```

But that defeats the purpose of the new AppArmor restriction in Ubuntu 24.04.

You can also create a new AppArmor profile for the Arduino IDE (that allows a non-root user to use the sandboxing in a specific application). If you copy the AppImage to /usr/local/bin/arduino (for example), you can create an AppArmor profile with a configuration file, for example, in /etc/apparmor.d/usr.local.bin.arduino, containing:

```
abi <abi/4.0>,
include <tunables/global>
profile arduino /usr/local/bin/arduino flags=(unconfined) {
  userns,
  include if exists <local/arduino>
}
```

and reloading all AppArmor profiles with:

```shell
sudo service apparmor reload
```

Now you can run the Arduino IDE without the sandboxing error.

You could also run the Arduino IDE with the --no-sandbox option, but that is, in my opinion, a potentially very bad idea.