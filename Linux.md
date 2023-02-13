# Linux Forensics

## OS and account information
### OS release information
`/etc/os-release`

### User accounts
`/etc/passwd`  
username : password information : user id (uid) : group id (gid) : description : home directory information : default shell  
```
user@machine$cat /etc/passwd| column -t -s :
root                  x  0      0      root                                /root                    /bin/bash
daemon                x  1      1      daemon                              /usr/sbin                /usr/sbin/nologin
.
.
games                 x  5      60     games                               /usr/games               /usr/sbin/nologin
.
.
ubuntu                x  1000   1000   Ubuntu                              /home/ubuntu             /bin/bash
```
The username is ubuntu, its password information field shows `x`, which signifies that the password information is stored in the `/etc/shadow` file.   

### Group Information
`/etc/group`  
```
user@machine$ cat /etc/group
.
.
adm:x:4:syslog,ubuntu
```
We can see that the user `ubuntu` belongs to the `adm` group, which has a password stored in the `/etc/shadow` file, signified by the `x` character. The gid is 4, and the group contains 2 users, Syslog, and ubuntu.

### Sudoers List
A Linux host allows only those users to elevate privileges to `sudo`, which are present in the Sudoers list.  
`/etc/sudoers`

### Login information
- `/var/log/btmp` : information about failed logins  
- `/var/log/wtmp` : historical data of logins  
They are binary file, which havet to be read using the `last` utility.
```
user@machine$ sudo last -f /var/log/wtmp
reboot   system boot  5.4.0-1029-aws   Tue Mar 29 17:28   still running
reboot   system boot  5.4.0-1029-aws   Tue Mar 29 04:46 - 15:52  (11:05)
reboot   system boot  5.4.0-1029-aws   Mon Mar 28 01:35 - 01:51 (1+00:16)

wtmp begins Mon Mar 28 01:35:10 2022
```
### Authentication logs
`/var/log/auth.log`  
We can use `tail`, `head`, `more` or `less` utilities to make it easier to read.
```
user@machine$ cat /var/log/auth.log |tail
Mar 29 17:28:48 tryhackme gnome-keyring-daemon[989]: The PKCS#11 component was already initialized
Mar 29 17:28:48 tryhackme gnome-keyring-daemon[989]: The SSH agent was already initialized
```

## System Configuration
### Hostname
`/etc/hostname`

### Timezone
`/etc/timezone`

### Network Configuration
`/etc/network/interfaces`
```
user@machine$ ip address show
```

### Active network connections
`netstat`
```
user@machine$ netstat -natp
.
.
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:5901          0.0.0.0:*               LISTEN      829/Xtigervnc
```

### Running processes
The `ps` utility shows details about the running processes.
```
user@machine$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         729  0.0  0.0   7352  2212 ttyS0    Ss+  17:28   0:00 /sbin/agetty -o -p -- \u --
```

### DNS information
`/etc/hosts`  
`/etc/resolv.conf` :The information about DNS servers  

## Persistence mechanisms
### Cron jobs
`/etc/crontab`
```
user@machine$ cat /etc/crontab 
.
.
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

### Service startup
Services can be set up in Linux that will start and run in the background after every system boot.  
`/etc/init.d` : We can check the contents of the directory by using the `ls` utility.  
```
user@machine$ ls /etc/init.d/
acpid       avahi-daemon      cups          hibagent           kmod             networking     pppd-dns                     screen-cleanup     unattended-upgrades
.
.
```

### .Bashrc
When a bash shell is spawned, it runs the commands stored in the `.bashrc` file. This file can be considered as a startup list of actions to be performed. Hence it can prove to be a good place to look for persistence.   
```
user@machine$ cat ~/.bashrc
```
System-wide settings are stored in `/etc/bash.bashrc` and `/etc/profile` files, so it is often a good idea to take a look at these files as well.  

## Evidence of Execution
### Sudo execution history
All the commands that are run on a Linux host using `sudo` are stored in `/var/log/auth.log`.  
We can use the `grep` utility to filter out only the required information from the auth log.
```
user@machine$ cat /var/log/auth.log* |grep -i COMMAND|tail
Mar 29 17:28:58 tryhackme pkexec[1618]: ubuntu: Error executing command as another user: Not authorized [USER=root] [TTY=unknown] [CWD=/home/ubuntu] [COMMAND=/usr/lib/update-notifier/package-system-locked]
Mar 29 17:49:52 tryhackme sudo:   ubuntu : TTY=pts/0 ; PWD=/home/ubuntu ; USER=root ; COMMAND=/usr/bin/cat /etc/sudoers
.
.
```