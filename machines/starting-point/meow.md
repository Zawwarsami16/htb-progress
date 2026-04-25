# Meow

**Tier:** Starting Point 0 · **OS:** Linux · **Difficulty:** very easy

## TL;DR

A single open port. Port 23. Telnet, with a banner that asks for a login. `root` with no password logs you in as root. There is no exploitation. The box is teaching you that telnet should not be on the open internet and that "no password" is still a credential.

## What I saw first

```
$ ping -c 1 10.129.194.131
64 bytes from 10.129.194.131: icmp_seq=1 ttl=63 time=3.38 ms

$ nmap -sV -sC -p 23 -Pn 10.129.194.131
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
```

TTL 63 means a Linux host one hop away over the VPN. The only open port is telnet. There is nothing else to enumerate.

## What I tried that did not work

Nothing, on the first attempt. This is the first box of the tier and it is designed to walk you straight to a shell.

## What worked

```
$ telnet 10.129.194.131
Meow login: root
Welcome to Ubuntu 20.04.2 LTS ...
root@Meow:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Meow:~# cat flag.txt
```

## Flag

`b40abdfe23665f766f9c61ecba8a4c19`

## What this taught me

The lesson is not telnet itself, which is obviously cooked. The lesson is that "no password" is a credential the system actually accepts. Whenever a service answers on a deprecated protocol, the first thing to try is the empty password, then the obvious defaults. The percentage of real-world systems that fall to this combination is higher than anyone would like to admit.
