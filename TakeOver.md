## CHALLENGE NAME : TakeOver
## ROOM LEVEL ; TYPE :  EASY ; WEB 
## INFO 

this is a web based challenge that focuses purely focuses on Subdomain enumeration only.


## GIVEN:




<img width="815" height="429" alt="Screenshot_From_2025-07-29_19-47-34" src="https://github.com/user-attachments/assets/b223ff45-5d49-420a-8806-4bb0df831097" />


take a good look at the hint , as per the room we are supposed to add the given IP address in the `/etc/hosts` path of our machine(vm or preferably the machine given in the room )
The hints that we can assume from the given room are:<br> 
	1) the company works on the topic space research and they write blogs about it(so they probably might have a separate blog site too  ) under a different subdomain name <br>
	2) "they are rebuilding their support" . now this may have 2 meanings either they have their support panel/page in their own website or it might be as a separate different website  under a different subdomain name
## APPROACH:

open your terminal and type the given command and add the given ip to the hosts folder like given below:

```
sudo nano /etc/hosts
```


<img width="815" height="429" alt="Screenshot_From_2025-07-29_19-57-44" src="https://github.com/user-attachments/assets/49c9270c-6599-49c3-816f-93531af041d5" />


now lets ping the given ip to test if it's working 

```
ping 10.10.198.62 -c5
PING 10.10.198.62 (10.10.198.62) 56(84) bytes of data.
64 bytes from 10.10.198.62: icmp_seq=1 ttl=61 time=470 ms
64 bytes from 10.10.198.62: icmp_seq=2 ttl=61 time=392 ms
64 bytes from 10.10.198.62: icmp_seq=3 ttl=61 time=415 ms
64 bytes from 10.10.198.62: icmp_seq=4 ttl=61 time=429 ms
64 bytes from 10.10.198.62: icmp_seq=5 ttl=61 time=358 ms

--- 10.10.198.62 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 358.365/412.870/470.176/37.346 ms     
```

Ok now , since it is pinging successfully lets move on to the enumeration phase . Here we should be performing a scan to check the list of open ports available by using rustscan or nmap.... but here i am going to do a simple nmap scan and here are its results

```
nmap -sCV 10.10.198.62
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-29 20:23 IST
Nmap scan report for futurevera.thm (10.10.198.62)
Host is up (0.41s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2d:02:ea:90:4f:68:f8:c9:13:c8:50:d2:ed:f1:81:75 (RSA)
|   256 b8:bc:c1:b0:86:ea:e2:1e:41:ad:f3:07:12:2a:64:81 (ECDSA)
|_  256 ad:3d:52:4b:a7:05:85:4d:79:68:3f:dd:1a:75:b8:57 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to https://futurevera.thm/
|_http-server-header: Apache/2.4.41 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
|_http-title: FutureVera
|_http-server-header: Apache/2.4.41 (Ubuntu)
| ssl-cert: Subject: commonName=futurevera.thm/organizationName=Futurevera/stateOrProvinceName=Oregon/countryName=US
| Not valid before: 2022-03-13T10:05:19
|_Not valid after:  2023-03-13T10:05:19
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 72.17 seconds
```

from this we can tell that there are three open ports <br> 
ssh     -    port 22 <br> 
http     -   port 80 <br> 
https   - port 443 <br> 
now lets check all the open services and see what we have..since this is a web based room we probably wont be using SSH in here and lets go and check the remaining two ports . <br> 
now close the terminal and go to the website and search for  `https://futurevera.thm` . I tried manually inspecting the code and found nothing . Then , I tired fuzzing the web endpoints with ffuf(you can also use gobuster instead of this)
to access it you can actually use the command  <br> 
i tried using http service it did'nt give me much useful information when compared to the https service
```
└─$ ffuf -w /usr/share/wordlists/dirb/common.txt -u https://10.10.198.62/ -H "Host: FUZZ.futurevera.thm" -fl 92      

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.198.62/
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Header           : Host: FUZZ.futurevera.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response lines: 92
________________________________________________

blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 364ms]
Blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 364ms]
support                 [Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 366ms]
Support                 [Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 366ms]
:: Progress: [4614/4614] :: Job [1/1] :: 107 req/sec :: Duration: [0:00:44] :: Errors: 0 ::
 
```

As we can see we get the subdomains such as `blog` and `support` i tried searching the files involved in their directories manually after adding their ip's too in the `/etc/hosts` directory
upon surfing through the page source i found nothing for a period of time and i actually wasted huge amount of time in that ...so after a while i started checking its certificate as i found it to be suspicious while entering the website as it asked for safety permissions . To my surprise , I found something in the `support`  website ie it's DNS name kept bugging me 

 <img width="925" height="633" alt="Screenshot_From_2025-07-29_22-10-32" src="https://github.com/user-attachments/assets/8a2fdfe6-0ea4-4f7b-af5c-d8872541c54c" />


So , i went to check the given DNS name and to my surprise the flag was present in the url as soon as i pressed the enter button

```
http://flag{find_it_yourself}.s3-website-us-west-3.amazonaws.com/
#this is not the actual flag folks
```
Looking forward to write more XD.
