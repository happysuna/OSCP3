- [ ] SSH-Audit
  ```bash
  git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
  ./ssh-audit.py $IP
  ```
- [ ] Tentare la connessione
	```bash
	ssh $IP
	```
  ```bash
	ssh user@$IP
	```
  ```bash
	ssh -i id_rsa user@$IP
	```
  ```bash
  ssh -v user@$IP -o PreferredAuthentications=password
  ```
	```bash
	ssh $IP -oKexAlgorithms=+ALGORITHM_NAME
	```
	```bash
	ssh $IP -oKexAlgorithms=+ALGORITHM_NAME -c CIPHER_NAME
	```
