# Bastard

## Reconnaissance

Per prima cosa eseguo una scansione utilizzando _nmapAutomator.sh_:

```text
Running all scans on 10.10.10.9

Host is likely running Windows

---------------------Starting Port Scan-----------------------

PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
49154/tcp open  unknown

---------------------Starting Script Scan-----------------------

PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Welcome to Bastard | Bastard
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

---------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports



PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
| vulners:
|   cpe:/a:microsoft:internet_information_services:7.5:
|     	CVE-2010-3972	10.0	https://vulners.com/cve/CVE-2010-3972
|     	SSV:20122	9.3	https://vulners.com/seebug/SSV:20122	*EXPLOIT*
|_    	CVE-2010-2730	9.3	https://vulners.com/cve/CVE-2010-2730
|_http-server-header: Microsoft-IIS/7.5
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


Running Vuln scan on all ports
This may take a while, depending on the number of detected services..


PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf:
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.9
|   Found the following possible CSRF vulnerabilities:
|     
|     Path: http://10.10.10.9:80/
|     Form id: user-login-form
|     Form action: /node?destination=node
|     
|     Path: http://10.10.10.9:80/user/register
|     Form id: user-register-form
|     Form action: /user/register
|     
|     Path: http://10.10.10.9:80/node?destination=node
|     Form id: user-login-form
|     Form action: /node?destination=node
|     
|     Path: http://10.10.10.9:80/user/password
|     Form id: user-pass
|     Form action: /user/password
|     
|     Path: http://10.10.10.9:80/user/
|     Form id: user-login
|     Form action: /user/
|     
|     Path: http://10.10.10.9:80/user
|     Form id: user-login
|_    Form action: /user
|_http-server-header: Microsoft-IIS/7.5
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

## Enumeration
Visito la pagina http://10.10.10.9/:

<p align="center">
  <img src="/Immagini/Windows-Box/Bastard/bastard-1.png"/>
</p>

Ispezionando la pagina http://10.10.10.9/CHANGELOG.txt scopro che la versione di _Drupal_ è la 7.54. Effettuando una rapida ricerca scopro che è vulnerabile (CVE-2018-7600).

Tramite l'utilizzo di _searchsploit_ trovo un file php che mi porta al seguente blog: https://www.ambionics.io/blog/drupal-services-module-rce.

Scopro quindi la possibilità di utilizzare la funzione _unserialize()_ per una serie di attacchi:
  - Escalation dei privilegi
  - SQL Injection
  - Esecuzione di codice da remoto

It links to this blog post. It seems to be a deserialization vulnerability that leads to Remote Code Execution (RCE). Looking at the code, it we see that it visit the path /rest_endpoint to conduct the exploit.

Guardando il codice php scopro che il path _/rest_endpoint_ conduce all'exploit. Ispezionando la pagina  http://10.10.10.9/rest mi imbatto in questo:

<p align="center">
  <img src="/Immagini/Windows-Box/Bastard/bastard-2.png"/>
</p>

Quindi utilizza il modulo _Services_ e per questo posso utilizzare l'exploit per accedere alla macchina.

## Exploitation
Provo a sfruttare la seguente vulnerabilità: https://github.com/pimps/CVE-2018-7600.

...


## Privilege Escalation

Ottengo una shell migliore con questo comando:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

...


## Credits

...
