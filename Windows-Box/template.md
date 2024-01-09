# Template

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con _nmapAutomator_.

```text
sudo nmapAutomator.sh --host 0.0.0.0 --type All
```

Il risultato della scansione è il seguente:

```text
risultato
```

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 0.0.0.0
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
risultato
```

Provo ad eseguire una scansione completa:

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 0.0.0.0
```

Il risultato della scansione è il seguente:

```text
risultato full
```

Provo ad eseguire una scansione udp
```text
sudo nmap -sU -O -p- -oA /usr/share/nmap/udp 0.0.0.0
```

Il risultato della scansione è il seguente:

```text
risultato udp
```

Aggiungo _template.htb_ al file _/etc/hosts_ e passo alla fase successiva.

## Enumeration
Avvio una scansione per verificare la presenza di vulnerabilità:

```text
nmap --script vuln -oA vuln 0.0.0.0
```

Successivamente, visito la pagina http://0.0.0.0/:

<p align="center">
  <img src="/Immagini/Windows-Box/Template/template-1.png"/>
</p>

Provo ad utilizzare il comando _gobuster_:

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://0.0.0.0/
```

Ottengo il seguente risultato:
```text
...
```

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://0.0.0.0/
```

```text
...
```

## Exploitation

...

## Privilege Escalation

Ottengo una shell migliore con questo comando:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Utilizzo Windows Exploit Suggester per identificare delle patch mancanti che possono permettere di scalare privilegi.

Per prima cosa aggiorno il database:

```text
python2 windows-exploit-suggester.py --update
```

Copio il risultato del comando _systeminfo_ eseguito sulla macchina vittima e lo salvo all'interno del file _systeminfo.txt_.

Poi lancio l'exploit suggester:

```text
python2 windows-exploit-suggester.py --database 2024-01-09-mssb.xls --systeminfo systeminfo.txt
```

...


## Credits

...
