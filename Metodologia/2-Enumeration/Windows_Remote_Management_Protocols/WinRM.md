# WinRM (5985/5986)
- [ ] WinRM Nmap
  ```bash
  nmap -sV -sC $IP -p5985,5986 --disable-arp-ping -n
  ```
- [ ] evil-winrm
  ```bash
  evil-winrm -i $IP -u user -p P455w0rD!
  ```
