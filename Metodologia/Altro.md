# Altro

## BloodHound
- [ ] Avviare neo4j
	```bash
	sudo neo4j start
	```
- [ ] Avviare BloodHound
	```bash
	BloodHound --no-sandbox
	```
- [ ] Scaricare le informazioni sull'Active Directory tramite SharpHound
	```bash
	SharpHound.exe -c DcOnly
	```
- [ ] Importare il file zip generato tramite SharpHound su BloodHound
