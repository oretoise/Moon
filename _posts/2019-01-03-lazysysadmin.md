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
| /index.html | Uhhh |
| /info.php | phpinfo() page |
| /javascript/ | 403 Error |
| /old/ | Empty directory |
| /phpmyadmin | PHPMyAdmin |
| /robots.txt | Reveals a few more directories: /old/, /test/, /TR2/, and /Backnode_files |
| /server-status | 403 |
| /test/ | Doesn't 403 like robots.txt indicates, but is empty. |
| /wordpress/ | A wordpress site. Screenshot below. |
| /wp/ | Empty |
