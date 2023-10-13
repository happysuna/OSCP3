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
Nmap scan report for 0.0.0.0
```

sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 0.0.0.0

```text
Nmap scan report for 0.0.0.0
```

sudo nmap -sU -O -p- -oA /usr/share/nmap/udp 0.0.0.0

```text
Nmap scan report for 0.0.0.0
```

## Enumeration

Innanzitutto, visito la pagina http://0.0.0.0/:

<p align="center">
  <img src="/Immagini/Linux-Box/Template/template-1.png" />
</p>

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
