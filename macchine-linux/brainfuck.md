# Brainfuck

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

* **Porta 22:** OpenSSH 7.2p2
* **Porta 25** smtp Postfix smtpd
* **Porta 110** pop3 Dovecot pop3d
* **Porta 143** imap Dovecot imapd
* **Porta 443** ssl/http nginx 1.10.0

```text
Nmap scan report for 10.10.10.17
Host is up (0.076s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 94d0b334e9a537c5acb980df2a54a5f0 (RSA)
|   256 6bd5dc153a667af419915d7385b24cb2 (ECDSA)
|_  256 23f5a333339d76d5f2ea6971e34e8e02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA PIPELINING RESP-CODES UIDL SASL(PLAIN) AUTH-RESP-CODE USER TOP
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: OK AUTH=PLAINA0001 Pre-login LITERAL+ ENABLE IMAP4rev1 more listed capabilities SASL-IR have post-login ID LOGIN-REFERRALS IDLE
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
| tls-nextprotoneg:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Linux 4.2 (92%), Linux 3.10 - 4.11 (90%), Linux 3.12 (90%), Linux 3.13 (90%), Linux 3.13 or 4.2 (90%), Linux 3.8 - 3.11 (90%)
No exact OS matches for host (test conditions non-ideal).
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ho aggiunto al file /etc/hosts/ i tre possibili hostname:
  * brainfuck.htb
  * www.brainfuck.htb
  * sup3rs3cr3t.brainfuck.htb

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 22 OpenSSH 7.2p2

La vulnerabilità trovata:
  * OpenSSH 2.3 < 7.7 - Username Enumeration

Questa vulnerabilità ci permette di fare username enumeration. Quindi scarico lo scpript da qui [CVE-2018-15473](https://github.com/Sait-Nuri/CVE-2018-15473) e lo eseguo passandogli una lista di username.

<p align="center">
  <img src="/immagini/immagini-macchine-linux/brainfuck/brainfuck-2.png" />
</p>

Nel frattempo che lo script esegue in background proseguo con gli altri servizi.

### 2) Porta 443 nginx 1.10.0

Per prima cosa visito il sito brainfuck.htb.

<p align="center">
  <img src="/immagini/immagini-macchine-linux/brainfuck/brainfuck-1.png" />
</p>

Questo è un sito WordPress e quindi ci sono delle vulnerabilità! Prima di lanciare uno scanner, provo a scovare qualche informazione utile nel certificato del sito e trovo questo indirizzo email:

<p align="center">
  <img src="/immagini/immagini-macchine-linux/brainfuck/brainfuck-3.png" />
</p>

Questo può essere utile successivamente.

Ora avvio lo scanner:

```text
wpscan --url https://brainfuck.htb --disable-tls-checks
```
 * — url: The URL of the blog to scan
 * — disable-tls-checks: Disables SSL/TLS certificate verification

Dalla scansione appena effettuata si evince che:
 * La versione di WordPress è la 4.7.3
 * Il plugin wp-support-plus-responsive-ticket-system contiene diverse vulnerabilità

Proseguo con una ricerca tramite searchsploit:

<p align="center">
  <img src="/immagini/immagini-macchine-linux/brainfuck/brainfuck-4.png" />
</p>

Cerco di sfruttare la seconda vulnerabilità della lista che mi consentirebbe di autenticarmi anche non conoscendo la password. Per testare questa vulnerabilità occorre conscrere un username valido:

```text
wpscan --url https://brainfuck.htb --disable-tls-checks --enumerate u
```

Il risultato della scansione è il seguente:

```text
[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] administrator
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

## Exploitation

### 1) ...

...
