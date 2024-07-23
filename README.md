# tryhackme-Pickle-Rick-CTF-Walkthrough
This is a detailed walkthrough for the CTF Pickle Rick on tryhackme.com.
First off, let's export the target IP as a variable.
Command: export ip=10.10.94.118
Next, let's run an nmap scan against the target.
Command: nmap -A -v $ip -oN nmap.txt
Command breakdown:
-A: This option enables aggressive scanning, combining several other options:
-T4: Sets the timing template to aggressive (faster scanning).
-F: Scans only the most common 100 ports.
-O: Enables OS detection.
--version-all: Attempts to determine the version of services running on open ports.
--script=default,http-enum,ssl-enum-ciphers: Runs default, HTTP enumeration, and SSL cipher enumeration scripts.
