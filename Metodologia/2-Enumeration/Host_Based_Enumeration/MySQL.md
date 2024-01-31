# MySQL (3306)
- [ ] MySQL Nmap
	```bash
  sudo nmap $IP -sV -sC -p3306 --script mysql*
	```
- [ ] Interaction with the MySQL Server
  ```bash
  mysql -u root -h $IP
  mysql -u root -pP4SSw0rd -h $IP
  ```
| **MySQL Command**   | **Description**   |
| --------------|-------------------|
| `mysql -u <user> -p<password> -h <IP address>` | Connect to the MySQL server. There should not be a space between the '-p' flag, and the password. |
| `show databases;` | Show all databases. |
| `use <database>;` | Select one of the existing databases. |
| `show tables;` | Show all available tables in the selected database. |
| `show columns from <table>;` | Show all columns in the selected database. |
| `select * from <table>;` | Show everything in the desired table. |
| `select * from <table> where <column> = "<string>";` | Search for needed string in the desired table. |
