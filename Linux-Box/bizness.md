# Bizness

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmapAutomator.sh --host 10.10.11.252 --type All
```

Il risultato della scansione è il seguente:

```text
Running all scans on 10.10.11.252

Host is likely running Linux


---------------------Starting Port Scan-----------------------



PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https



---------------------Starting Script Scan-----------------------



PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey:
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp  open  http     nginx 1.18.0
|_http-title: Did not follow redirect to https://bizness.htb/
|_http-server-header: nginx/1.18.0
443/tcp open  ssl/http nginx 1.18.0
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=UK
| Not valid before: 2023-12-14T20:03:40
|_Not valid after:  2328-11-10T20:03:40
|_http-title: Did not follow redirect to https://bizness.htb/
|_ssl-date: TLS randomness does not represent time
|_http-server-header: nginx/1.18.0
| tls-alpn:
|_  http/1.1
| tls-nextprotoneg:
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel




---------------------Starting Full Scan------------------------



PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
443/tcp   open  https
43529/tcp open  unknown



Making a script scan on extra ports: 43529



PORT      STATE SERVICE    VERSION
43529/tcp open  tcpwrapped

```

Aggiungo _bizness.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Successivamente, visito la pagina http://0.0.0.0/:

<p align="center">
  <img src="/Immagini/Linux-Box/Template/template-1.png"/>
</p>

Provo ad utilizzare il comando _gobuster_:

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://0.0.0.0/
```

Ottengo il seguente risultato:
```text
...
```

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u https://bizness.htb/
```

```text
[10:13:32] 302 -    0B  - /accounting  ->  https://bizness.htb/accounting/
[10:13:46] 302 -    0B  - /catalog  ->  https://bizness.htb/catalog/
[10:13:47] 404 -  762B  - /common/
[10:13:47] 404 -  779B  - /common/config/db.ini
[10:13:47] 302 -    0B  - /common  ->  https://bizness.htb/common/
[10:13:47] 404 -  780B  - /common/config/api.ini
[10:13:49] 302 -    0B  - /content  ->  https://bizness.htb/content/
[10:13:49] 302 -    0B  - /content/  ->  https://bizness.htb/content/control/main
[10:13:49] 302 -    0B  - /content/debug.log  ->  https://bizness.htb/content/control/main
[10:13:50] 200 -   34KB - /control
[10:13:50] 200 -   34KB - /control/
[10:13:53] 200 -   11KB - /control/login
[10:13:54] 302 -    0B  - /error  ->  https://bizness.htb/error/
[10:13:54] 302 -    0B  - /example  ->  https://bizness.htb/example/
[10:13:59] 302 -    0B  - /images  ->  https://bizness.htb/images/
[10:14:00] 302 -    0B  - /index.jsp  ->  https://bizness.htb/control/main
[10:14:25] 200 -   21B  - /solr/admin/file/?file=solrconfig.xml
[10:14:25] 200 -   21B  - /solr/admin/
[10:14:25] 302 -    0B  - /solr/  ->  https://bizness.htb/solr/control/checkLogin/
```

## Exploitation

...

## Privilege Escalation

Ottengo una shell migliore con questo comando:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

...


## Credits

...
