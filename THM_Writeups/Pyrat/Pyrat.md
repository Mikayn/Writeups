# Pyrat

## **Challenge Information:**

**Link:** [tryhackme.com/room/pyrat](http://tryhackme.com/room/pyrat)
**Difficulty:** Easy
**Category:** Boot-to-Root
**Description:**
- Name: Pyrat
- Description: Test your enumeration skills on this boot-to-root machine.
**Scenario:** 
![{7B64F933-AC99-40C5-8D27-43A8EB0C2316}.png](Images/7B64F933-AC99-40C5-8D27-43A8EB0C2316.png)

## TLDR

A Python socket server on port 8000 directly interprets user input, allowing a reverse shell as `www-data`. Credentials for user `think` are found in a `.git/config` file due to password reuse. Root is achieved by fuzzing a hidden `admin` endpoint on the same socket server and brute-forcing the password with a custom script.

## Initial Reconnaissance

### Nmap Scan: 

Command: `nmap -A -v <IP> -oN nmapresult.txt` 

![{1041BAE7-6AF3-4C49-BC7C-EDFF747FC0F9}.png](Images/1041BAE7-6AF3-4C49-BC7C-EDFF747FC0F9.png)

Scan showed two ports open: **22 (SSH)** and **8000** (identified by nmap as a Python-based server). Port 8000 became the primary focus first due to the lack of credentials for SSH login.

### Port 8000 — Python Socket Server

![{ABA49BAB-1956-4F11-BC74-64A7C4197B1D}.png](Images/ABA49BAB-1956-4F11-BC74-64A7C4197B1D.png)

I tried `netcat` but did not see anything so i thought connection was not being successful and i tried `telnet` next. I was wrong and nc does work. The server is a Python socket which interprets the input silently, which is what confused me. 

```bash
telnet <IP> 8000
```

![{5454F575-5A73-4B05-B2E4-9F2897ABC96A}.png](Images/5454F575-5A73-4B05-B2E4-9F2897ABC96A.png)

### Initial Access

Since nmap identified the service is on Python, I tried some Python reverse shells but i got an error. The error made me realize that the input might be passed directly to an interpreter. 

![{5C972C0A-1D17-447C-BBF7-16AD7F32F923}.png](Images/5C972C0A-1D17-447C-BBF7-16AD7F32F923.png)

I tried other reverse shells just in case, but got the same result. So I removed the `python -c` at the start.

![image.png](Images/image.png)

And I got a different error `name 's' is not defined`, which showed that Python was directly being interpreted by the server. My listener also briefly connected before being closed, probably because of that error. I knew i was on the right track.

The payload worked and I was in as `www-data`. 

![{9953FBDF-651D-47D1-A1BA-439D6A9C17B3}.png](Images/9953FBDF-651D-47D1-A1BA-439D6A9C17B3.png)

## Shell as www-data

I did some enumeration and found a user `think`. `www-data` has very little privilege so that was my target first before trying for root. 

![{648EDE34-82C8-452A-886D-5B291E6ECD64}.png](Images/648EDE34-82C8-452A-886D-5B291E6ECD64.png)

Upon further manual enumeration, i found `.git` in the `/opt/dev` directory. The hint in tryhackme says *Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials.* So, I knew that `think` password would have to be somewhere here.   

![{A711F49E-8887-4A75-B29E-27297B402715}.png](Images/A711F49E-8887-4A75-B29E-27297B402715.png)

Looking around, I found `think`'s github password in the `config` file. Password reuse is a common trope and i was in the system as `think`. Thats why you always use different passwords kids.  
```bash
cat /opt/dev/.git/config
```

![{E8B4BD5A-6EE2-4CE2-ABEB-D82271E24182}.png](Images/E8B4BD5A-6EE2-4CE2-ABEB-D82271E24182.png)

## Shell as think

### Investigating Git History

```bash
su think
```

![{6CAC2CE5-76FF-4168-BF7F-DB2B27694BB0}.png](Images/6CAC2CE5-76FF-4168-BF7F-DB2B27694BB0.png)

Got the user flag ez. Now time to root the machine. Thankfully, I had a hint on tryhackme which said `A subsequent exploration yields valuable insights into the application's older version`. Since git was present, I decided to check commits first. 

I also remembered this in index. So the name of the older version was `pyrat.py.old`. 

![{CACF76F4-7417-4FF7-BB40-4DDBF83F07C0}.png](Images/CACF76F4-7417-4FF7-BB40-4DDBF83F07C0.png)

Found the previous versions by checking the changes. Did not have to look through commits so thanks author. 

![{C45E45A2-6FA1-4E23-B7C5-78964AA0F2E8}.png](Images/C45E45A2-6FA1-4E23-B7C5-78964AA0F2E8.png)

(I accidently let the machine expire at this point, so the IP used will be different). 

### Fuzzing the Endpoint

`pyrat.py.old` showed the logic at the admin endpoint.
![{9DF66300-A0EE-4D5D-B637-24789872DA17}.png](Images/9DF66300-A0EE-4D5D-B637-24789872DA17.png)

If the socket is admin, and “shell” is passed as input, it spawns a shell? Thats what I understood and I sent `admin` to the socket, and it prompted for a password.

```bash
telnet  8000
admin
```

### Brute-Forcing the Password

So this was the bruteforce the hint was talking about. But I cant bruteforce directly since I need to type “admin” for the password prompt. So a custom script was needed. 

![{D0FBD14C-8136-47D7-8039-1F0270C83FC3}.png](Images/D0FBD14C-8136-47D7-8039-1F0270C83FC3.png)

After a LOT of trial and error, i got this script to work. The script establishes a new connection per password, sends `admin`, then tests each entry from `rockyou.txt`. Its not the most efficient approach, but its reliable.

![{D739BDF8-2BED-40DC-94DE-14F7899DDB8E}.png](Images/D739BDF8-2BED-40DC-94DE-14F7899DDB8E.png)

Running the script, we see that the password is `abc123`. 

### Getting Root

![{714B71EE-CAB9-4012-9FAD-9220B697D01F}.png](Images/714B71EE-CAB9-4012-9FAD-9220B697D01F.png)

```bash
telnet  8000
admin
abc123
```

And I got the root flag. 

![{32B1F102-E0D4-4139-8695-050B50154119}.png](Images/32B1F102-E0D4-4139-8695-050B50154119.png)

--- 

## Exploitation Chain Summary

| Step | Action | Result |
|------|--------|--------|
| 1 | Nmap scan | Port 8000 identified as Python socket server |
| 2 | Send raw Python payload to port 8000 | Reverse shell as `www-data` |
| 3 | Read `.git/config` in `/opt/dev` | Credentials for user `think` |
| 4 | SSH as `think` with reused password | User flag |
| 5 | Review old commit (`pyrat.py.old`) | Discovered hidden `admin` endpoint |
| 6 | Brute-force admin password with custom script | Password: `abc123` |
| 7 | Authenticate to admin endpoint | Root shell + root flag |
