# Reconnaissance

## Pre engagement
- [ ] Inserimento del IP target nella variabile $IP
	```bash
	export $IP=x.x.x.x
	```

## Metodologia Generale
- [ ] Aggiunta dell'host al file **/etc/hosts**
- [ ] Eseguire delle scansioni **nmap**
- [ ] Per ogni porta TCP/UDP aperta:
	- [ ] Trovare il servizio e la versione
	- [ ] Trovare i bug legati al servizio
	- [ ] Trovare problemi nella configurazione
	- [ ] Salavare ogni messaggio di errore
	- [ ] Scoprire tutti i path URL
- [ ] Per ogni servizio scoperto effettuare delle ricerche su **Google** e utilizzare **searchsploit**

## Banner Grabbing
- [ ] nc -v $IP \<PORT\>
- [ ] telnet $IP \<PORT\>

## Network & Port scanning
Se non si è a conoscenza di host attivi, potrebbe essere necessario effettuare una scansione sull'intera subnet.

### Scansioni sulla subnet con
- [ ] List scan
	```bash
	nmap -sL -oN nmap/listScan 10.x.x.x/xx
	```
- [ ] Ping scan (run it with privileges)
	```bash
	nmap -sn -oN nmap/pingScan 10.x.x.x/xx
	```
- [ ] Ricerca di informazioni sugli host (name, logged-in user, MAC)
	```bash
	nbtscan -r 10.x.x.x.x
	```
- [ ] Hosts discovery
	```bash
	netdiscover -r 10.x.x.x/24
	```
- [ ] smbtree

### Scansioni sul singolo host
- [ ] Scansione Iniziale:
	```bash
	sudo nmap -sC -sV -O -oA /usr/share/nmap/initial $IP
	```
- [ ] Scansione Completa:
	```bash
	sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full $IP
	```
- [ ] Fast TCP port scan:
	```bash
	nmap -F -T4 -oN nmap/fastTCPScan $IP
	```
- [ ] Simple TCP port scan
	```bash
	nmap -p- -T4 -oN nmap/ezTCPScan $IP
	```
- [ ] Simple UDP port scan
	```bash
	nmap -sU -n -p- -T4 -oN nmap/ezUDPScan $IP
	```
- [ ] Aggressive scan
	```bash
	nmap -A -T4 -px,y,z -v -oN nmap/aggressiveScan $IP
	```
- [ ] Version detection sulle porte TCP
	```bash
	nmap -sV --reason -O -p- $IP
	```
- [ ] Version detection sulle porte UDP
	```bash
	nmap -sU -sV -n $IP
	```
- [ ] Capire se l'host è vulnerabile all'heartbleed?
	```bash
	nmap --script ssl-heartbleed $IP
	```
- [ ] Version/OS detection
	```bash
	nmap -v --dns-server \<DNS\> -sV --reason -O --open -Pn $IP
	```
- [ ] Identificazione si servizi sconosciuti
	```bash
	amap -d $IP \<PORT\>
	```
- [ ] Full vulnerability scanning
	```bash
	nmap -sS -sV --script=/path/to/your/vulnscan.nse -oN nmap/vulnScan $IP
	```
