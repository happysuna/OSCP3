# SMB (139)
- [ ] Check Default Configuration
	```bash
	cat /etc/samba/smb.conf | grep -v "#\|\;"
	```
- [ ] Restart Samba
	```bash
	sudo systemctl restart smbd
	```
- [ ] Check for share
	```bash
	smbmap -H $IP
	```
	```bash
	crackmapexec smb $IP --shares -u '' -p ''
	```
	```bash
	smbclient -N -L //$IP
	```
	```bash
	smbclient -N -L \\\\$IP\\
	```
- [ ] Connect to SMB share
	```bash
	smbclient //$IP/SHARE_NAME
	```
	```bash
	smbclient \\\\$IP\\SHARE_NAME$
	```
	```bash
	smbclient //$IP/C$ -U site.com/username%Password
	```
	```bash
	smbclient //$IP/transfer -U username --pw-nt-hash 820d6348590813116884101357197052 -W site.com
	```
- [ ] Execute Commands
	```bash
	smb: \> !ls
	smb: \> !cat prep-prod.txt
	```
- [ ] Check Connections
	```bash
	smbstatus
	```
- [ ] SMB Nmap
	```bash
	sudo nmap $IP -sV -sC -p139,445
	```
- [ ] rpcclient
	```bash
	rpcclient -U "" $IP
	```
	| **rpcclient Query**   | **Description**   |
	| --------------|-------------------|
	| `srvinfo` | Server information |
	| `enumdomains` | Enumerate all domains that are deployed in the network |
	| `querydominfo` | Provides domain, server, and user information of deployed domains |
	| `netshareenumall` | Enumerates all available shares |
	| `netsharegetinfo <share>` | Provides information about a specific share |
	| `enumdomusers` | Enumerates all domain users |
	| `queryuser <RID>` | Provides information about a specific user |
- [ ] Brute Forcing User RIDs
	```bash
	for i in $(seq 500 1100);do rpcclient -N -U "" $IP -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
	```
- [ ] Impacket - Samrdump.py
	```bash
	samrdump.py $IP
	```
- [ ] [Enum4Linux-ng](https://github.com/cddmp/enum4linux-ng)
	```bash
	cd enum4linux-ng
	pip3 install -r requirements.txt
	./enum4linux-ng.py $IP -A
	```
