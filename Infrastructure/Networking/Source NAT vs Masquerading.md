---
created: 2024-04-02T08:56
updated: 2024-04-02T09:00
tags:
  - Networking
  - NAT
related link: https://serverfault.com/a/119374
---

> [!INFO] In a nutshell
> If your outgoing interface has a address that is static, then you don't need to use MASQ and can use SNAT which will be a little faster since it doesn't need to figure out what the external IP is every time.


> **Source NAT changes the source address in IP [header](http://en.wikipedia.org/wiki/IPv4#Header) of a packet.** It may also change the **source** port in the TCP/UDP headers. The typical usage is to change the a private (rfc1918) address/port into a public address/port for packets leaving your network.
> 
> [...]
> 
> **Masquerading is a special form of Source NAT where the source address is unknown at the time the rule is added to the tables in the kernel.** If you want to allow hosts with private address behind your firewall to access the Internet and the external address is variable (DHCP) this is what you need to use. Masquerading will modify the source IP address and port of the packet to be the primary IP address assigned to the outgoing interface. 
