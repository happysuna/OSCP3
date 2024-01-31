# RDP (3389)
- [ ] RDP Nmap
  ```bash
  nmap -sV -sC $IP -p3389 --script rdp*
  ```
  ```bash
  nmap -sV -sC $IP -p3389 --packet-trace --disable-arp-ping -n
  ```
- [ ] RDP Security Check
  ```bash
  sudo cpan
  git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
  ./rdp-sec-check.pl $IP
  ```
- [ ] Initiate an RDP Session
  ```bash
  xfreerdp /u:user /p:"P455w0rd!" /v:$IP
  ```
