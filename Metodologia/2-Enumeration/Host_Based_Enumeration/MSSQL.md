# MSSQL (1443)
- [ ] MSSQL Nmap
	```bash
  sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 $IP
	```
- [ ] Connecting with Mssqlclient.py (Impacket)
  ```bash
  python3 mssqlclient.py Administrator@$IP
  python3 mssqlclient.py Administrator@$IP -windows-auth
  ```
- [ ] Connecting with sqsh
  ```bash
  sqsh -S <IP> -U <Username> -P <Password> -D <Database>
  sqsh -S <IP> -U .\\<Username> -P <Password> -D <Database> #Windows
  ```
- [ ] Metasploit
  ```text
  use scanner/mssql/mssql_ping
  ```
- [ ] [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server)
