# Drive

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
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 10.10.10.117
```

Il risultato della scansione è il seguente:

```text
risultato full
```

Provo ad eseguire una scansione udp
```text
sudo nmap -sU -O -p- -oA /usr/share/nmap/udp 10.10.10.117
```

Il risultato della scansione è il seguente:

```text
risultato udp
```

## Enumeration

Innanzitutto, visito la pagina http://0.0.0.0/:

<p align="center">
  <img src="/Immagini/Linux-Box/Template/template-1.png" />
</p>

```text
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://0.0.0.0/
```

```text
dirsearch -u http://0.0.0.0/
```

Provo quindi ad inserire gli script elencati e uno di questi (_listfiles.php_) produce un output interessante:

```text
...
```

## Exploitation

...

## Privilege Escalation

...


## Credits

...
