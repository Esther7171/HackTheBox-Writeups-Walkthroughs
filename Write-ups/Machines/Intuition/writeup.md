# <div align="center">[Intuition]()</div>
### <div align="center">Linux · Hard</div>

<div align="center">
  <img src="https://labs.hackthebox.com/storage/avatars/464537cc0d3e9962fc598767bff7b1f1.png" height="200" align="center"></img>
</div>

## Scanning The Network
```
nmap 10.10.11.15 -sV -Pn -A
```
![image](https://github.com/user-attachments/assets/46570475-061d-4702-8515-a7040f40888a)

* ### According to Scan results:
* #### SSH is open.
* #### HTTP is open

## Web Enumeration
* ### Lets Add the domain to /etc/hosts
```
echo "10.10.11.15 comprezzor.htb" | sudo tee -a /etc/hosts
```
* ### After adding we can start our Enumeration.
```
ffuf -c -u http://comprezzor.htb -H 'Host: FUZZ.comprezzor.htb' -ic -w ~/wordlists/seclists/current/Discovery/DNS/subdomains-top1million-110000.txt -r -fs 3408
```
