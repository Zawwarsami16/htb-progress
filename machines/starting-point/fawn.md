# Fawn

**Tier:** Starting Point 0 · **OS:** Linux · **Difficulty:** very easy

## TL;DR

A single open port. Port 21. vsftpd 3.0.3 with anonymous FTP enabled. Log in as `anonymous`, get the flag from the listing. nmap's `ftp-anon` script does the work for you before you even open an FTP client.

## What I saw first

```
$ nmap -sV -sC -p 21 10.129.1.14
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed
| -rw-r--r-- 1 0 0 32 Jun 14  2021 flag.txt
```

The NSE script `ftp-anon` already logged in as anonymous and listed the directory. There is a `flag.txt`, thirty-two bytes, last modified June 2021. The box is telling you everything before you even open the FTP client.

## What I tried that did not work

Nothing. The box is two minutes from start to flag.

## What worked

```
$ curl -u anonymous:anonymous ftp://10.129.1.14/flag.txt
035db21c881520061c53e0536e44f815
```

`anonymous` with any password works. Empty password also works.

## Flag

`035db21c881520061c53e0536e44f815`

## What this taught me

Two patterns worth keeping for the rest of the tier and for real engagements.

`nmap -sC` runs the default script set, which includes `ftp-anon`, `ssh-hostkey`, `http-title`, and a long list of other low-friction probes. Always run it on the first scan. It saves you from manually probing the obvious things.

Anonymous FTP existed for decades and still pops up on machines that someone forgot. Port 21 open in the wild is worth `anonymous:anonymous` every time. Half the time you get nothing. The other half you get a directory listing that hands you the next move.
