# Enumeration

## SSH (22)
- [ ] Ricavare la versione
- [ ] Tentare la connessione
	```bash
	ssh $IP
	```
	```bash
	ssh $IP -oKexAlgorithms=+ALGORITHM_NAME
	```
	```bash
	ssh $IP -oKexAlgorithms=+ALGORITHM_NAME -c CIPHER_NAME
	```

## HTTP/HTTPS (80/443)
- [ ] Visitare la pagina
- [ ] Cercare informazioni utili
	- [ ] Source Page
- [ ] Utilizzare Burpsuite
- [ ] Directory Buster
	- [ ] _ffuf_ (Vedere guida)
	- [ ] _dirbuster_
	- [ ] _gobuster_
	```bash
	gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://$IP
	```
	- [ ] _dirsearch_
	```bash
	dirsearch -u http://$IP
	```

## SMB (139)
- [ ] Ricavare la versione
- [ ] Check for anonymous share
	```bash
	smbmap -H $IP
	```
	```bash
	smbclient -L \\\\$IP\\
	```
- [ ] Connect to SMB share
	```bash
	smbclient \\\\$IP\\SHARE_NAME$
	```
	```bash
	smbclient //$IP/C$ -U site.com/username%Password
	```
	```bash
	smbclient //$IP/transfer -U username --pw-nt-hash 820d6348590813116884101357197052 -W site.com
	```

## DNS (53)
- [ ] Ricerca di nomi di dominio asocciati all'host
	```bash
	whois $IP
	```
	```bash
	nslookup
	> server $IP
	Default server: ...
	Address: ...
	> $IP
	...
	```
- [ ] Ricerca IP e authoritative servers
	```bash
	nslookup domain.com
	```
- [ ] Resolve DNS
	```bash
	host website.com
	nslookup website.com
	```
- [ ] Ricerca name servers
	```bash
	host -t ns domain.com
	```
- [ ] Ricerca mail servers
	```bash
	host -t mx domain.com
	```
- [ ] Zone transfer possibile?
	```bash
	nmap --script dns-zone-transfer -p 53 domain.com
	```
- [ ] Richiesta zone transfer
	```bash
	host -l domain.name dns.server

	dig axfr @dns-server domain.name

	dnsrecon -d domain.com -t axfr

	dnsrecon -d $IP -D /usr/share/wordlists/dnsmap.txt -t std --xml ouput.xml
	````
- [ ] Sono presenti vulnerabilità conosciute?
	- [ ] Ricerca su https://www.exploit-db.com/
	- [ ] Ricerca su https://www.cvedetails.com/
	- [ ] Ricerca su https://nvd.nist.gov/
	- [ ] Ricerca su google
		```bash
		site:github.com *Service version.release*
		```

## Linux Enumeration
- [ ] Enumerazione di informazioni locali utilizzando [LinEnum](https://github.com/rebootuser/LinEnum):
```bash
./LinEnum.sh
```
- [ ] Enumerazione di informazioni locali utilizzando [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS):
```bash
./linpeas.sh
```
- [ ] Ricerca di possibili exploits on Linux utilizzando [Linux Exploit Suggester](https://github.com/jondonas/linux-exploit-suggester-2):
	```bash
	./linux-exploit-suggester-2.pl
	```
