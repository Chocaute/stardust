---
created: 2024-03-12T15:58
updated: 2024-03-12T16:07
tags:
  - Debian
  - Linux
related link: https://wiki.archlinux.org/title/File_permissions_and_attributes#Changing_permissions
---

> [!NOTE] About "chmod"
> [chmod](https://en.wikipedia.org/wiki/chmod "wikipedia:chmod") is a command in Linux and other Unix-like operating systems that allows to change the permissions (or access mode) of a file or directory.

### Text method

To change the permissions — or _access mode_ — of a file, use the _chmod_ command in a terminal. Below is the command's general structure:

```shell
chmod who=permissions filename
```

Where `who` is any from a range of letters, each signifying who is being given the permission. They are as follows:

- `u`: the [user](https://wiki.archlinux.org/title/User "User") that owns the file.
- `g`: the [user group](https://wiki.archlinux.org/title/User_group "User group") that the file belongs to.
- `o`: the **o**ther users, i.e. everyone else.
- `a`: **a**ll of the above; use this instead of typing `ugo`.

The permissions are `r` read, `w` write, and `x` execute.

Now have a look at some examples using this command. Suppose you became very protective of the Documents directory and wanted to deny everybody but yourself, permissions to read, write, and execute (or in this case search/look) in it:

Before: `drwxr-xr-x 6 archie web 4096 Jul 5 17:37 Documents`

```shell
chmod g= Documents
chmod o= Documents
```

After: `drwx------ 6 archie web 4096 Jul 6 17:32 Documents`

Here, because you want to deny permissions, you do not put any letters after the `=` where permissions would be entered. Now you can see that only the owner's permissions are `rwx` and all other permissions are `-`.

This can be reverted with:

Before: `drwx------ 6 archie web 4096 Jul 6 17:32 Documents`

```shell
chmod g=rx Documents
chmod o=rx Documents
```

After: `drwxr-xr-x 6 archie web 4096 Jul 6 17:32 Documents`

In the next example, you want to grant read and execute permissions to the group, and other users, so you put the letters for the permissions (`r` and `x`) after the `=`, with no spaces.

You can simplify this to put more than one `who` letter in the same command, e.g:

```shell
chmod go=rx Documents
```

> [!NOTE]
> It does not matter in which order you put the `_who_` letters or the permission letters in a `chmod` command: you could have `chmod go=rx file` or `chmod og=xr file`. It is all the same.

Now let us consider a second example, suppose you want to change a `foobar` file so that you have read and write permissions, and fellow users in the group `web` who may be colleagues working on `foobar`, can also read and write to it, but other users can only read it:

Before: `-rw-r--r-- 1 archie web 5120 Jun 27 08:28 foobar`

```shell
chmod g=rw foobar
```

After: `-rw-rw-r-- 1 archie web 5120 Jun 27 08:28 foobar`

This is exactly like the first example, but with a file, not a directory, and you grant write permission (just so as to give an example of granting every permission).

#### Text method shortcuts

The _chmod_ command lets add and subtract permissions from an existing set using `+` or `-` instead of `=`. This is different from the above commands, which essentially re-write the permissions (e.g. to change a permission from `r--` to `rw-`, you still need to include `r` as well as `w` after the `=` in the _chmod_ command invocation. If you missed out `r`, it would take away the `r` permission as they are being re-written with the `=`. Using `+` and `-` avoids this by adding or taking away from the _current_ set of permissions).

Let us try this `+` and `-` method with the previous example of adding write permissions to the group:

Before: `-rw-r--r-- 1 archie web 5120 Jun 27 08:28 foobar`

```shell
chmod g+w foobar
```

After: `-rw-rw-r-- 1 archie web 5120 Jun 27 08:28 foobar`

Another example, denying write permissions to all (**a**):

Before: `-rw-rw-r-- 1 archie web 5120 Jun 27 08:28 foobar`

```shell
chmod a-w foobar
```

After: `-r--r--r-- 1 archie web 5120 Jun 27 08:28 foobar`

A different shortcut is the special `X` mode: this is not an actual file mode, but it is often used in conjunction with the `-R` option to set the executable bit only for directories, and leave it unchanged for regular files, for example:

```shell
chmod -R a+rX ./data/
```

#### Copying permissions

It is possible to tell _chmod_ to copy the permissions from one class, say the owner, and give those same permissions to group or even all. To do this, instead of putting `r`, `w`, or `x` after the `=`, put another _who_ letter. e.g:

Before: `-rw-r--r-- 1 archie web 5120 Jun 27 08:28 foobar`

```shell
chmod g=u foobar
```

After: `-rw-rw-r-- 1 archie web 5120 Jun 27 08:28 foobar`

This command essentially translates to "change the permissions of group (`g=`), to be the same as the owning user (`=u`). Note that you cannot copy a set of permissions as well as grant new ones e.g.:

```shell
chmod g=wu foobar
```

In that case _chmod_ throw an error.

### Numeric method

_chmod_ can also set permissions using numbers.

Using numbers is another method which allows you to edit the permissions for all three owner, group, and others at the same time, as well as the setuid, setgid, and sticky bits. This basic structure of the code is this:

```shell
chmod xxx filename
```

Where `xxx` is a 3-digit number where each digit can be anything from 0 to 7. The first digit applies to permissions for owner, the second digit applies to permissions for group, and the third digit applies to permissions for all others.

In this number notation, the values `r`, `w`, and `x` have their own number value:

r=4
w=2
x=1

To come up with a 3-digit number you need to consider what permissions you want owner, group, and all others to have, and then total their values up. For example, if you want to grant the owner of a directory read write and execution permissions, and you want group and everyone else to have just read and execute permissions, you would come up with the numerical values like so:

- Owner: `rwx`=4+2+1=7
- Group: `r-x`=4+0+1=5
- Other: `r-x`=4+0+1=5

```shell
chmod 755 filename
```

This is the equivalent of using the following:

```shell
chmod u=rwx filename
chmod go=rx filename
```

To view the existing permissions of a file or directory in numeric form, use the [stat(1)](https://man.archlinux.org/man/stat.1) command:

```shell
stat -c %a filename
```

Where the %a option specifies output in numeric form.

Most directories are set to `755` to allow reading, writing and execution to the owner, but deny writing to everyone else, and files are normally `644` to allow reading and writing for the owner but just reading for everyone else; refer to the last note on the lack of `x` permissions with non executable files: it is the same thing here.

To see this in action with examples consider the previous example that has been used but with this numerical method applied instead:

Before: `-rw-r--r-- 1 archie web 5120 Jun 27 08:28 foobar`

```shell
chmod 664 foobar
```

After: `-rw-rw-r-- 1 archie web 5120 Jun 27 08:28 foobar`

If this were an executable the number would be `774` if you wanted to grant executable permission to the owner and group. Alternatively if you wanted everyone to only have read permission the number would be `444`. Treating `r` as 4, `w` as 2, and `x` as 1 is probably the easiest way to work out the numerical values for using `chmod xxx filename`, but there is also a binary method, where each permission has a binary number, and then that is in turn converted to a number. It is a bit more convoluted, but here included for completeness.

Consider this permission set:

-rwxr-xr--

If you put a 1 under each permission granted, and a 0 for every one not granted, the result would be something like this:

-rwxrwxr-x
 111111101

You can then convert these binary numbers:

000=0	    100=4
001=1	    101=5
010=2	    110=6
011=3	    111=7

The value of the above would therefore be 775.

Consider we wanted to remove the writable permission from group:

-rwxr-xr-x
 111101101

The value would therefore be 755 and you would use `chmod 755 _filename_` to remove the writable permission. You will notice you get the same three digit number no matter which method you use. Whether you use text or numbers will depend on personal preference and typing speed. When you want to restore a directory or file to default permissions e.g. read and write (and execute) permission to the owner but deny write permission to everyone else, it may be faster to use `chmod 755/644 _filename_`. However if you are changing the permissions to something out of the norm, it may be simpler and quicker to use the text method as opposed to trying to convert it to numbers, which may lead to a mistake. It could be argued that there is not any real significant difference in the speed of either method for a user that only needs to use _chmod_ on occasion.

You can also use the numeric method to set the `setuid`, `setgid`, and `sticky` bits by using four digits.

setuid=4
setgid=2
sticky=1

For example, `chmod 2777 filename` will set read/write/executable bits for everyone and also enable the `setgid` bit.