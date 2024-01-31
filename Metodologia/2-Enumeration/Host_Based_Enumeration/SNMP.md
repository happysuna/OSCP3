# SNMP (161/162)
- [ ] SNMPwalk
	```bash
  snmpwalk -v2c -c public $IP
	```
- [ ] OneSixtyOne
	```bash
  sudo apt install onesixtyone
  onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt $IP
	```
- [ ] Braa
	```bash
  sudo apt install braa
  braa <community string>@<IP>:.1.3.6.*   # Syntax
  braa public@$IP:.1.3.6.*  # Esempio
	```
