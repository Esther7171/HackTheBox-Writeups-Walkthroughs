# <div align="center">Lame</div>
<p align="center">Linux · Easy </p>

<div align="center">
    <img align-item="center" width="200"  src="https://labs.hackthebox.com/storage/avatars/fb2d9f98400e3c802a0d7145e125c4ff.png" alt="Material Bread logo">
</div>

# Task 1:
## How many of the nmap top 1000 TCP ports are open on the remote host?
```
4
```
# Task 2:
## What version of VSFTPd is running on Lame?
```
2.3.4
```
# Task 3:
## There is a famous backdoor in VSFTPd version 2.3.4, and a Metasploit module to exploit it. Does that exploit work here?
```
no
```
# Task 4:
## What version of Samba is running on Lame? Give the numbers up to but not including "-Debian".
```
3.0.20
```
# Task 5:
## What 2007 CVE allows for remote code execution in this version of Samba via shell metacharacters involving the SamrChangePassword function when the "username map script" option is enabled in smb.conf?
```
CVE-2007-2447
```
# Task 6:
## Exploiting CVE-2007-2447 returns a shell as which user?
```
root
```
# Task 7:
## Submit the flag located in the makis user's home directory.
```
979d0b01c45b33323f5c84980bafa032
```

# Task 8:
## Submit the flag located in root's home directory.

```
b021c9a53a054900abcecf87e74be2f6
```
# Task 9:
## We'll explore a bit beyond just getting a root shell on the box. While the official writeup doesn't cover this, you can look at 0xdf's write-up for more details. With a root shell, we can look at why the VSFTPd exploit failed. Our initial nmap scan showed four open TCP ports. Running netstat -tnlp shows many more ports listening, including ones on 0.0.0.0 and the boxes external IP, so they should be accessible. What must be blocking connection to these ports?
```
firewall
```

# Task 10:
## When the VSFTPd backdoor is trigger, what port starts listening?
```
6200
```
# Task 11:
## When the VSFTPd backdoor is triggered, does port 6200 start listening on Lame?
```
Yes
```
# Task 11:
```
yes
```
