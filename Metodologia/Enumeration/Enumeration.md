# Enumeration

## DNS (UDP/TCP 53)
- [ ] Find domain names for host
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
- [ ] Find IP and authoritative servers
	```bash
	nslookup domain.com
	```
- [ ] Resolve DNS
	```bash
	host website.com
	nslookup website.com
	```
- [ ] Find name servers
	```bash
	host -t ns domain.com
	```
- [ ] Find mail servers
	```bash
	host -t mx domain.com
	```
- [ ] Is DNS zone transfer possible?
	```bash
	nmap --script dns-zone-transfer -p 53 domain.com
	```
- [ ] Request zone transfer
	```bash
	host -l domain.name dns.server

	dig axfr @dns-server domain.name

	dnsrecon -d domain.com -t axfr
	````
- [ ] dnsrecon -d $IP -D /usr/share/wordlists/dnsmap.txt -t std --xml ouput.xml
- [ ] Any known vulnerability?
	- [ ] Check https://www.exploit-db.com/
	- [ ] Check https://www.cvedetails.com/
	- [ ] Check https://nvd.nist.gov/
	- [ ] Check on google
		```bash
		site:github.com *Service version.release*
		```

## Linux Enumeration
- [ ] Enumerate local information using [LinEnum](https://github.com/rebootuser/LinEnum):
```bash
./LinEnum.sh
```
- [ ] Enumerate local information using [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS):
```bash
./linpeas.sh
```
- [ ] Find possible exploits on Linux using [Linux Exploit Suggester](https://github.com/jondonas/linux-exploit-suggester-2):
	```bash
	./linux-exploit-suggester-2.pl
	```
