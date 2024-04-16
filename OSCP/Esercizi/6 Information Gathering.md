## 6.3.3. Port Scanning with Nmap

```bash
sudo nmap -v -p 25 -sS $IP/24 -T5 -oG 25-port-open.txt
cat 25-port-open.txt | grep open
```

```bash
nmap -p 43 --script whois-ip -oG whois_scan_results.txt $IP/24
```

```bash
nmap -F -T5 $IP
```

```bash
nmap -sT -T4 -p 50000-60000 $IP
```

```bash
nmap -p80 --script http-title --open -v $IP
```

## 6.3.4. SMB Enumeration

```bash
nmap -v -p 139,445 -oG smb.txt $IP-RANGE
cat smb.txt | grep open | wc -l
```

```bash
xfreerdp /u:student /p:lab /v:$IP
net view \\dc01 /all
```

```bash
nmap -v -p 139,445 -oG smb.txt $IP-RANGE
cat smb.txt | grep open | awk '{print $2}' > open_smb_host.txt
while IFS= read -r ip_address; do enum4linux "$ip_address"; done < open_smb_host.txt
```
## 6.3.5. SMTP Enumeration

```bash
sudo nmap -v -p 25 -sS $IP/24 -T5 -oG 25-port-open.txt
nc -nv $IP 25
VRFY root
VRFY idontexist
```
## 6.3.6. SNMP Enumeration

```bash
snmpwalk -v2c -c public $IP -Oa
```
