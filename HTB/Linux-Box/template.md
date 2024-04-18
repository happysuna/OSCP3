# Template

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

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

Successivamente, visito la pagina http://0.0.0.0/:

<p align="center">
  <img src="/Immagini/Linux-Box/Template/template-1.png"/>
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

...


## Credits

...
