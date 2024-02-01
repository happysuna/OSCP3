# FTP (20/21)
- [ ] Anonymous Login
  ```bash
  ftp 10.129.14.136
  ```
  ```text
  anonymous:anonymous
  ```
- [ ] Recursive Listing
  ```bash
  ftp> ls -R
  ```
- [ ] Download All Available Files
  ```bash
  wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
  ```
- [ ] Nmap FTP Scripts
  ```bash
  sudo nmap --script-updatedb
  ```
  ```bash
  find / -type f -name ftp* 2>/dev/null | grep scripts
  ```
  ```bash
  sudo nmap -sV -p21 -sC -A 10.10.10.10
  ```
- [ ] Service Interaction
  ```bash
  nc -nv 10.129.14.136 21
  ```
  ```bash
  telnet 10.129.14.136 21
  ```
  ```bash
  openssl s_client -connect 10.129.14.136:21 -starttls ftp
  ```
