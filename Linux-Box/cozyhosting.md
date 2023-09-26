# CozyHosting

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.230
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 22**: OpenSSH 8.9p1
* **Porta 80**:  nginx 1.18.0

```text
Nmap scan report for cozyhosting.htb (10.10.11.230)
Host is up (0.050s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home
|_http-server-header: nginx/1.18.0 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/22%OT=22%CT=1%CU=33530%PV=Y%DS=2%DC=I%G=Y%TM=650D966
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=105%GCD=1%ISR=10A%TI=Z%CI=Z%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=
OS:FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

Innanzitutto, vedo cosa c'è sulla porta 80:

<p align="center">
  <img src="/Immagini/Linux-Box/CozyHosting/cozyhosting-1.png" />
</p>

Aggiungo il nome di dominio _cozyhosting.htb_ al file _/etc/hosts/_.

Utilizzo _dirb_, dal quale ottengo il seguente risultato:

```text
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Sep 22 09:35:26 2023
URL_BASE: http://cozyhosting.htb/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://cozyhosting.htb/ ----
+ http://cozyhosting.htb/admin (CODE:401|SIZE:97)                                                                                                                   
+ http://cozyhosting.htb/error (CODE:500|SIZE:73)                                                                                                                   
+ http://cozyhosting.htb/index (CODE:200|SIZE:12706)                                                                                                                
+ http://cozyhosting.htb/login (CODE:200|SIZE:4431)                                                                                                                 
+ http://cozyhosting.htb/logout (CODE:204|SIZE:0)                                                                                     

-----------------
END_TIME: Fri Sep 22 09:39:52 2023
DOWNLOADED: 4612 - FOUND: 5

```

Ma non trovo nulla di interessante. Quindi provo con _dirsearch_:

```text
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/kali/.dirsearch/reports/cozyhosting.htb/-_23-09-22_11-03-50.txt

Error Log: /home/kali/.dirsearch/logs/errors-23-09-22_11-03-50.log

Target: http://cozyhosting.htb/

[11:03:50] Starting:
[11:03:58] 200 -    0B  - /Citrix//AccessPlatform/auth/clientscripts/cookies.js
[11:04:02] 400 -  435B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[11:04:03] 400 -  435B  - /a%5c.aspx
[11:04:04] 200 -    5KB - /actuator/env
[11:04:04] 200 -  634B  - /actuator
[11:04:04] 200 -   15B  - /actuator/health
[11:04:04] 200 -   10KB - /actuator/mappings
[11:04:04] 200 -   48B  - /actuator/sessions
[11:04:04] 200 -  124KB - /actuator/beans
[11:04:04] 401 -   97B  - /admin
[11:04:26] 200 -    0B  - /engine/classes/swfupload//swfupload.swf
[11:04:26] 200 -    0B  - /engine/classes/swfupload//swfupload_f9.swf
[11:04:26] 500 -   73B  - /error
[11:04:26] 200 -    0B  - /examples/jsp/%252e%252e/%252e%252e/manager/html/
[11:04:27] 200 -    0B  - /extjs/resources//charts.swf
[11:04:30] 200 -    0B  - /html/js/misc/swfupload//swfupload.swf
[11:04:31] 200 -   12KB - /index
[11:04:35] 200 -    4KB - /login
[11:04:35] 200 -    0B  - /login.wdm%2e
[11:04:35] 204 -    0B  - /logout
[11:04:50] 400 -  435B  - /servlet/%C0%AE%C0%AE%C0%AF

Task Completed

```

Visito la pagina http://cozyhosting.htb/actuator/sessions:

<p align="center">
  <img src="/Immagini/Linux-Box/CozyHosting/cozyhosting-2.png"/>
</p>

Da qui riesco a recuperare l'id di sessione dell'utente _kanderson_. Utilizzando _burp_ riesco ad entrare nella pagina http://cozyhosting.htb/admin con l'id di sessione appena scoperto:

<p align="center">
  <img src="/Immagini/Linux-Box/CozyHosting/cozyhosting-3.png"/>
</p>







## Exploitation

...

## Privilege Escalation

...
