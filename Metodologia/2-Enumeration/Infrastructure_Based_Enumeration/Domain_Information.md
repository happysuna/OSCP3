# Domain Information
- [ ] Controllare il certificato SSL
- [ ] [crt.sh](https://crt.sh/)
	```bash
	curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
	```
	```bash
	curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
	```
- [ ] Shodan - IP List
  ```bash
  for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
  for i in $(cat ip-addresses.txt);do shodan host $i;done
	```
- [ ] DNS Records
  ```bash
  dig any inlanefreight.com
  ```
