---
created: 2024-02-22T14:50
updated: 2024-02-22T14:51
tags:
  - Linux
  - Ubuntu
  - Notifications
---
https://askubuntu.com/a/187034

There are a bunch of other cool features with [`notify-send`](http://manpages.ubuntu.com/manpages/precise/man1/notify-send.1.html)

We can run a command and make it display in the notification:

```
notify-send <title> <`command`>
notify-send Date "`date`"
notify-send Disk "`df / -H`"
```

We can use icons with the notifications

```
notify-send -i <icon> <Message>
notify-send -i face-wink "Hello! January"
```

Really annoying pop up

```
notify-send  -t 0 "Bringing down the system"
```

and

```
notify-send <title> <message>
notify-send "who am i" "I am January"
```
