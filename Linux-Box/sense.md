# Sense

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.230
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 80**:  lighttpd 1.4.35
* **Porta 443**:  lighttpd 1.4.35

```text
Nmap scan report for 10.10.10.60
Host is up (0.050s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
443/tcp open  ssl/http lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Not valid before: 2017-10-14T19:21:35
|_Not valid after:  2023-04-06T19:21:35
|_http-title: Login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: specialized|general purpose
Running (JUST GUESSING): Comau embedded (90%), OpenBSD 4.X (87%)
OS CPE: cpe:/o:openbsd:openbsd:4.0
Aggressive OS guesses: Comau C4G robot control unit (90%), OpenBSD 4.0 (87%)
No exact OS matches for host (test conditions non-ideal)
```

## Enumeration

Innanzitutto, vedo cosa c'è sulla porta 80:

<p align="center">
  <img src="/Immagini/Linux-Box/Sense/sense-1.png" />
</p>

Mi ritrovo un form per l'accesso a un firewall pfSense.

Provo ad utilizzare il comando _dirsearch_:

```text
Target: https://10.10.10.60/

[09:58:17] Starting:
[09:58:25] 403 -  345B  - /.dep.inc
[09:58:26] 403 -  345B  - /.env~
[09:58:30] 403 -  345B  - /.gitignore~
[09:58:33] 403 -  345B  - /.inc
[09:58:42] 403 -  345B  - /.ssh/id_rsa.key~
[09:58:42] 403 -  345B  - /.ssh/id_rsa~
[09:58:42] 403 -  345B  - /.ssh/know_hosts~
[09:58:43] 403 -  345B  - /.ssh/id_rsa.priv~
[09:58:43] 403 -  345B  - /.ssh/id_rsa.pub~
[09:58:45] 403 -  345B  - /.travis.yml~
[09:59:12] 403 -  345B  - /_config.inc
[09:59:27] 403 -  345B  - /admin/includes/configure.php~
[09:59:57] 403 -  345B  - /adovbs.inc
[10:00:00] 403 -  345B  - /app/config/database.yml~
[10:00:04] 403 -  345B  - /auth.inc
[10:00:07] 403 -  345B  - /backup.inc
[10:00:08] 403 -  345B  - /backups.inc
[10:00:14] 403 -  345B  - /cabal.project.local~
[10:00:18] 200 -  271B  - /changelog.txt
[10:00:19] 301 -    0B  - /classes  ->  https://10.10.10.60/classes/
[10:00:22] 403 -  345B  - /common.inc
[10:00:23] 403 -  345B  - /conf.inc.php~
[10:00:24] 403 -  345B  - /config.inc
[10:00:24] 403 -  345B  - /config.inc.php~
[10:00:24] 403 -  345B  - /config.inc~
[10:00:25] 403 -  345B  - /config.local.php~
[10:00:25] 403 -  345B  - /config.php.inc~
[10:00:25] 403 -  345B  - /config.php.inc
[10:00:25] 403 -  345B  - /config.php~
[10:00:25] 403 -  345B  - /config/database.yml~
[10:00:25] 403 -  345B  - /config/db.inc
[10:00:26] 403 -  345B  - /config/config.inc
[10:00:26] 403 -  345B  - /configuration.inc.php~
[10:00:26] 403 -  345B  - /config/settings.inc
[10:00:26] 403 -  345B  - /configuration~
[10:00:26] 403 -  345B  - /configuration.php~
[10:00:26] 403 -  345B  - /config~
[10:00:27] 403 -  345B  - /conf~
[10:00:28] 403 -  345B  - /connect.inc
[10:00:30] 301 -    0B  - /css  ->  https://10.10.10.60/css/
[10:00:32] 403 -  345B  - /database.yml~
[10:00:32] 403 -  345B  - /database.inc
[10:00:33] 403 -  345B  - /db.inc
[10:00:33] 403 -  345B  - /database_credentials.inc
[10:00:34] 403 -  345B  - /debug.inc
[10:00:39] 403 -  345B  - /dump.inc
[10:00:40] 200 -    7KB - /edit.php
[10:00:46] 200 -    1KB - /favicon.ico
[10:00:53] 403 -  345B  - /globals.inc
[10:00:58] 403 -  345B  - /includes/adovbs.inc
[10:00:58] 301 -    0B  - /includes  ->  https://10.10.10.60/includes/
[10:00:59] 403 -  345B  - /includes/bootstrap.inc
[10:00:59] 403 -  345B  - /includes/configure.php~
[10:00:59] 200 -  329B  - /index.html
[10:00:59] 403 -  345B  - /inc/config.inc
[10:00:59] 200 -    7KB - /index.php
[10:01:00] 403 -  345B  - /index.inc
[10:01:00] 403 -  345B  - /index.php~
[10:01:00] 200 -    7KB - /index.php/login/
[10:01:01] 403 -  345B  - /index~
[10:01:02] 301 -    0B  - /installer  ->  https://10.10.10.60/installer/
[10:01:02] 403 -  345B  - /install.inc
[10:01:04] 301 -    0B  - /javascript  ->  https://10.10.10.60/javascript/
[10:01:08] 200 -    7KB - /license.php
[10:01:10] 403 -  345B  - /localsettings.php~
[10:01:34] 403 -  345B  - /php.ini~
[10:01:52] 403 -  345B  - /revision.inc
[10:01:53] 403 -  345B  - /sample.txt~
[10:01:57] 403 -  345B  - /settings.php~
[10:02:04] 403 -  345B  - /sql.inc
[10:02:06] 200 -    7KB - /status.php
[10:02:11] 200 -    7KB - /system.php
[10:02:12] 200 -    7KB - /system-users.txt
[10:02:14] 301 -    0B  - /themes  ->  https://10.10.10.60/themes/
[10:02:31] 403 -  345B  - /wp-config.inc
[10:02:31] 403 -  345B  - /wp-config.php.inc
[10:02:31] 403 -  345B  - /wp-config.php~
[10:02:34] 200 -  384B  - /xmlrpc.php

Task Completed
```

La prima pagina che ispeziono è https://10.10.10.60/changelog.txt:

<p align="center">
  <img src="/Immagini/Linux-Box/Sense/sense-2.png" />
</p>

Da qui scopro che c'è una vulnerabilità del firewall che non è stata ancora risolta e che potrei sfruttare.


Successivamente, visito la pagina https://10.10.10.60/system-users.txt:

<p align="center">
  <img src="/Immagini/Linux-Box/Sense/sense-3.png" />
</p>

Quindi potrebbe esistere un utente _Rohit_ con password di default. Facendo una rapida ricerca scopro che una password di default per questo firewall potrebbe essere _pfsense_. Quindi provo ad autenticarmi con le credenziali appena scoperte:

<p align="center">
  <img src="/Immagini/Linux-Box/Sense/sense-4.png" />
</p>

Dalla dashboard scopro che la versione del firewall è la 2.1.3 ed effettuando una ricerca tramite _searchesploit_ scopro che è vulnerabile al command injection.

```text
-------------------------------------------------------------------------------------------- ---------------------------------
Exploit Title                                                                              |  Path
-------------------------------------------------------------------------------------------- ---------------------------------
pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command Injection                             |  php/webapps/43560.py
-------------------------------------------------------------------------------------------- ---------------------------------
```

Seguo le indicazioni del file python scaricato e prima di lanciare lo script avvio un listener in ascolto sulla porta 4444...



## Privilege Escalation

...
