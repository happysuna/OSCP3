### SMTP (25/587/465)

<p align="center">
  <img src="/Immagini/Enumeration/enumeration-2.png" />
</p>

| **Command**   | **Description**   |
| --------------|-------------------|
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client. |
| `HELO` | The client logs in with its computer name and thus starts the session. |
| `MAIL FROM` | The client names the email sender. |
| `RCPT TO` | The client names the email recipient. |
| `DATA` | The client initiates the transmission of the email. |
| `RSET` | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY` | The client checks if a mailbox is available for message transfer. |
| `EXPN` | The client also checks if a mailbox is available for messaging with this command. |
| `NOOP` | The client requests a response from the server to prevent disconnection due to time-out. |
| `QUIT` | The client terminates the session. |

- [ ] Local SMTP Configuration
	```bash
	cat /etc/bind/named.conf.local
	```
- [ ] Telnet - HELO/EHLO
	```bash
   telnet $IP 25
	```
  ```bash
   >HELO prova.com
	```
  ```bash
   >EHLO prova
	```
- [ ] Telnet - VRFY
	```bash
	telnet $IP 25
	```
  ```bash
   >VRFY root
	```
  ```bash
   >VRFY user123
	```
- [ ] SMTP Nmap
	```bash
	sudo nmap $IP -sC -sV -p25
	```
  ```bash
	sudo nmap $IP -p25 --script smtp-open-relay -v
	```
  ```bash
	nmap --script smtp-enum-users.nse $IP
	```
  ```bash
	smtp-user-enum -M VRFY -U smtp-usernames.txt -D prova.com -t $IP
	```
- [ ] Metasplit
  ```text
  use auxiliary/scanner/smtp/smtp_enum
  ```
