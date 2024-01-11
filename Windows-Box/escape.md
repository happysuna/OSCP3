# Escape

## Reconnaissance

Per prima cosa eseguo una scansione completa con _nmapAutomator_.

```text
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

## Enumeration
Parto da SMB:


```text
smbclient -L 10.10.11.202
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-1.png"/>
</p>

Provo a vedere cosa c'è nello share Public:

```text
smbclient -L //10.10.11.202/Public -N
```

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-2.png"/>
</p>

Quindi scarico il pdf e lo visualizzo:

<p align="center">
  <img src="/Immagini/Windows-Box/Escape/escape-3.png"/>
</p>


...

## Exploitation

...

## Privilege Escalation

...


## Credits

...
