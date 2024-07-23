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
-v: enables verbose output, it displays additional information about the scanning process, open ports, and services. 
$ip: This calls the variable storing the target IP.
-oN nmap.txt: this stores the output of the scan in a normal format to nmap.txt

![nmap_result](https://github.com/user-attachments/assets/737586aa-2213-453e-90af-a0da1e937a7e)
This reveals two open ports, open ssh on 22 and an http web server on 80. Let's see what we can find on the browser. When we view the page source code, the username is revealed:
![pagesource](https://github.com/user-attachments/assets/a7e04774-c2b6-4594-9e8c-b82db000b0d1)
