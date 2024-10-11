---
created: 2024-04-02T09:47
updated: 2024-06-12T09:42
tags:
  - Syncthing
  - Systemd
related link: https://docs.syncthing.net/users/autostart.html#using-systemd
---
### How to set up as a system service

1. Create the user who should run the service, or choose an existing one.

2. *(Skip if your distribution package already installs these files, see above.)* From the git repository copy the `syncthing@.service` file into the load path of the system instance.

3. Enable and start the service. Replace `myuser` with the Syncthing user after the `@`:
```shell
systemctl enable syncthing@myuser.service
systemctl start syncthing@myuser.service
```

### How to set up as a user service

1. Create the user who should run the service, or choose an existing one. Probably this will be your own user account.

2. *(Skip if your distribution package already installs these files, see above.)* From the git repository copy the `syncthing.service` file into the load path of the user instance. To do this without root privileges you can just use this folder under your home directory: `~/.config/systemd/user/`.

3. Enable and start the service:
```shell
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```

> [!warning]-  If your home directory is encrypted with eCryptfs on Debian/Ubuntu
> You will need to make the change described in Ubuntu bug 1734290. Otherwise the user service will not start, because by default, systemd checks for user services before your home directory has been decrypted.

Automatic start-up of systemd user instances at boot (before login) is possible through systemd’s “lingering” function, if a system service cannot be used instead. Refer to the enable-linger command of `loginctl` to allow this for a particular user.

### Checking the service status

To check if Syncthing runs properly you can use the `status` subcommand. To check the status of a system service:
```shell
systemctl status syncthing@myuser.service
```

To check the status of a user service:
```shell
systemctl --user status syncthing.service
```
