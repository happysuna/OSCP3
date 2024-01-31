# NFS (111)
- [ ] NFS Nmap
	```bash
	sudo nmap $IP -p111,2049 -sV -sC
	```
	```bash
	sudo nmap --script nfs* $IP -sV -p111,2049
	```
- [ ] Show Available NFS Shares
	```bash
	showmount -e $IP
	```
- [ ] Mounting NFS Share
	```bash
	mkdir target-NFS
	sudo mount -t nfs $IP:/ ./target-NFS/ -o nolock
	cd target-NFS
	tree .
	```
- [ ] List Contents with Usernames & Group Names
	```bash
	ls -l mnt/nfs/
	```
- [ ] List Contents with UIDs & GUIDs
	```bash
	ls -n mnt/nfs/
	```
- [ ] Unmounting NFS Share
	```bash
	sudo umount ./target-NFS
	```
