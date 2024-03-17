---
created: 2024-03-12T16:09
updated: 2024-03-12T16:13
tags:
  - Linux
  - Debian
  - Cybersecurity
related link: https://linuxconfig.org/how-to-manage-acls-on-linux
---
> **ACLs are a second level of discretionary permissions and may override the standard ugo/rwx ones.** 
> 
> They can an grant a better granularity in setting access to a file or a directory, for example by giving or denying access to a specific user that is neither the file owner, nor in the group owner.

The first thing you have to do, if you want to take advantage of ACLs is to make sure that the filesystem you want to use them on, has been mounted with the ‘acl’ option. To verify the latter you can run the ‘tune2fs -l’ command, passing the partition as argument. 

```shell
sudo tune2fs -l /dev/mmcblk0p2 | grep "Default"
```

If your filesystem has not been mounted with the ‘acl’ option, you can re-mount it giving the needed option:

```shell
mount -o remount -o acl /dev/sda1
```

However, notice that the mount options set this way, will not be persistent, and will not survive a reboot. If you want to obtain persistence, you have to modify the filesystem mount options in /etc/fstab, assigning the ‘acl’ option statically.

Another thing we need, is to install the `acl` package. This package contains various ACLs utilities like the `getfacl` and `setfacl` programs.

## A test case

Let’s see what ACLs can do for us. First we will create a file named text.cfg and we will give it as an argument to the `getfacl` command. Let’s see what the output of this command shows:

```shell
touch text.cfg && getfacl text.cfg
```

As you can see, since we didn’t set any ACL permission on the file, the command just displays the standard permissions values, plus the file owner and the group owner, both having read and write permissions. Now let’s imagine we want to give a specific user (I will create this user on purpose and call him `dummy` ), a specific set of privileges on the file. We will just have to run:

```shell
setfacl -m u:dummy:rw text.cfg
```

Let’s analyze the command: first we have, of course, the name of the program `setfacl`, which is pretty self-explanatory, then we passed the `-m` option (short for `--modify`) which allows us to change the ACLs of a file, then the permission descriptions `u:dummy:rw`.

We have three ‘sections’ divided by colons: in the first one, the `u` stands for user, specifying that we want to set the ACLs for a specific user. It could have been a `g` for group, or an `o` for `others`. In the second section we have the name of the user whom we want to set the permissions for, and in the third, the permissions to assign.

Finally, the name of the file on which we want to apply the permissions.

If we now try to run the ‘getfacl’ command, we can see that its output reflects the changes we made:

```shell
getfacl text.cfg
`````

---

An entry has been added for the `dummy` user, showing the permissions we assigned to him. Other than that, if you notice, also an entry for `mask` has appeared. What does it stand for ? The mask associated with an ACL limits the set of permissions that can be assigned on the file for the named groups and users and for the group owner, but has no effect on the permissions for the file owner and the `other` permission group.

In this case, only reading and writing permissions could be assigned with setfacl command. Of course we can change this option, using `setfacl` program itself:

```shell
setfacl -m mask:r text.cfg
```

With the command above, we set the mask to allow only reading permissions. Let’s check the output of `getfacl` now:

```shell
getfacl text.cfg
```

As you can see, not only the changes we made to the mask is now reported, but also the effective permissions for the group owner and the named user `dummy` are showed. Although the group owner and the `dummy` user have reading and writing permissions on the file, by changing the mask, we have effectively limited their permissions to read only. As the output of the command shows, they now are only allowed to read the file.

Other than explicitly changed with the command above, the ACLs mask also gets automatically re-calculated when we assign or change permissions with setfacl (unless the -n option is specified). Let’s demonstrate that: we will change the permissions of the `dummy` user to `rwx` and then check the getfacl output:

```shell
setfacl -m u:dummy:rwx text.cfg && getfacl text.cfg
```

As you can see the mask got re-calculated and it now reflects the maximum permissions present for the named user `dummy`. Obviously, since now no previously set permissions are higher than the mask, there is no need for showing the `#effective` permission status.

You can also use ACL to completely deny access to a file for a specific named user or group. For example, by running:

```shell
setfacl -m u:dummy:- text.cfg
```

we effectively deny all privileges to the `dummy` user on the text.cfg file.

---

## Default ACLs

The `default` ACL is a specific type of permission assigned to a directory, that doesn’t change the permissions of the directory itself, but makes it so that the specified ACLs are set by default on all the files created inside of it. Let’s demonstrate it: first we are going to create a directory and assign `default` ACL to it by using the `-d` option:

```shell
mkdir test && setfacl -d -m u:dummy:rw test
```

now, we can examine the output of the getfacl for that directory:

```shell
getfacl test
```

The `default` permissions has been assigned correctly. Now we can verify them by creating a file inside of the test directory and checking its permissions by running getfacl:

```shell
touch test/file.cfg && getfacl test/file.cfg
```

As expected, the file has been created automatically receiving the ACLs permissions specified above.

When you want to erase all the ACLs set, you can always run the setfacl with the `-b` option.

This tutorial covers the main aspects of ACLs, and of course there is a lot more about them to know, so I suggest, as always, to read the manual for a more in-deep knowledge. By now just remember that if you want to remove all the ACLs permissions assigned to a file, you just have to run `setfacl` with the `-b` (short for `--remove-all`) option.