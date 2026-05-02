# Pyrat

## **Challenge Information:**

**Link:** [tryhackme.com/room/pyrat](http://tryhackme.com/room/pyrat)

**Description:**

- Name: Pyrat
- Description: Test your enumeration skills on this boot-to-root machine.

**Scenario:** 

![{7B64F933-AC99-40C5-8D27-43A8EB0C2316}.png](Images/7B64F933-AC99-40C5-8D27-43A8EB0C2316.png)

## Initial Reconnaissance

Nmap Scan: 

Command: `nmap -A -v <IP> -oN nmapresult.txt` 

![{1041BAE7-6AF3-4C49-BC7C-EDFF747FC0F9}.png](Images/1041BAE7-6AF3-4C49-BC7C-EDFF747FC0F9.png)

Website at Port 8000

![{ABA49BAB-1956-4F11-BC74-64A7C4197B1D}.png](Images/ABA49BAB-1956-4F11-BC74-64A7C4197B1D.png)

I tried `netcat` but did not see anything so i thought connection was not being successful and i tried `telnet` next.  I was wrong and nc does work. If you put some python commands, it will output it, but I did not know then. 

![{5454F575-5A73-4B05-B2E4-9F2897ABC96A}.png](Images/5454F575-5A73-4B05-B2E4-9F2897ABC96A.png)

We know the server is on python from the nmap scan. So i tried some python reverse shells but i got an error.  

![{5C972C0A-1D17-447C-BBF7-16AD7F32F923}.png](Images/5C972C0A-1D17-447C-BBF7-16AD7F32F923.png)

I thought that revshell was not working so i tried another one, but got the same result. Then I thought since the server is on Python, our input is being parsed directly to the python interpreter. So the `python -c` at the start would not be needed.

![image.png](Images/image.png)

And I got a different error `name 's' is not defined`, which showed that the input was being passed to the interpreter. My listener also briefly connected before being closed, probably because of that error. I knew i was on the right track so i tried some another reverse shell command. 

This command worked, and viola i was in as `www-data`. 

![{9953FBDF-651D-47D1-A1BA-439D6A9C17B3}.png](Images/9953FBDF-651D-47D1-A1BA-439D6A9C17B3.png)

## Shell as www-data

I did some enumeration and found a user `think`. `www-data` has very little privilege so that was my target first before trying for root. 

![{648EDE34-82C8-452A-886D-5B291E6ECD64}.png](Images/648EDE34-82C8-452A-886D-5B291E6ECD64.png)

Upon further manual enumeration, i found `.git` in the `/opt/dev` directory. The hint in tryhackme says *Delving into the directories, the author uncovers a well-known folder that provides a user with access to credentials.* So, I knew that `think` password would have to be somewhere here.   

![{A711F49E-8887-4A75-B29E-27297B402715}.png](Images/A711F49E-8887-4A75-B29E-27297B402715.png)

Looking around, I found `think`'s github password in the `config` file. Password reuse is a common trope and i was in the system as `think`. Thats why you always use different passwords kids.  

![{E8B4BD5A-6EE2-4CE2-ABEB-D82271E24182}.png](Images/E8B4BD5A-6EE2-4CE2-ABEB-D82271E24182.png)

## Shell as think

![{6CAC2CE5-76FF-4168-BF7F-DB2B27694BB0}.png](Images/6CAC2CE5-76FF-4168-BF7F-DB2B27694BB0.png)

Got the user flag ez. Now time to root the machine. Thankfully, I had a hint on tryhackme which said `A subsequent exploration yields valuable insights into the application's older version`. Since git was present, I decided to check commits first. 

I also remembered this in index. So the name of the older version was `pyrat.py.old`. 

![{CACF76F4-7417-4FF7-BB40-4DDBF83F07C0}.png](Images/CACF76F4-7417-4FF7-BB40-4DDBF83F07C0.png)

Found the previous versions by checking the changes. Did not have to look through commits so thanks author. 

![{C45E45A2-6FA1-4E23-B7C5-78964AA0F2E8}.png](Images/C45E45A2-6FA1-4E23-B7C5-78964AA0F2E8.png)

(I accidently let the machine expire at this point, so the IP used will be different). 

![{9DF66300-A0EE-4D5D-B637-24789872DA17}.png](Images/9DF66300-A0EE-4D5D-B637-24789872DA17.png)

If the socket is admin, and “shell” is passed as input, it spawns a shell? Thats what I understood but I didnt know what to do next. So i checked tryhackme for the hint. 

*Exploring possible endpoints using a custom script, the user can discover a special endpoint and ingeniously expand their exploration by fuzzing passwords*. So I had to fuzz for endpoints and then bruteforce the login. 

Since the code talked about admin, i assumed the endpoint was also `admin`.  And it was lol. 

Putting `admin` in the telnet prompts for a password. So this was the bruteforce the hint was talking about. But I cant bruteforce directly since I need to type “admin” for the password prompt. So a custom script was needed. 

![{D0FBD14C-8136-47D7-8039-1F0270C83FC3}.png](Images/D0FBD14C-8136-47D7-8039-1F0270C83FC3.png)

After a LOT of trial and error, i got this script to work. Its not very efficient since it establishes the connection for every password in `rockyou`, but this was the best I could come up with. 

![{D739BDF8-2BED-40DC-94DE-14F7899DDB8E}.png](Images/D739BDF8-2BED-40DC-94DE-14F7899DDB8E.png)

Running the script, we see that the password is `abc123`. 

![{714B71EE-CAB9-4012-9FAD-9220B697D01F}.png](Images/714B71EE-CAB9-4012-9FAD-9220B697D01F.png)

And I got root. 

![{32B1F102-E0D4-4139-8695-050B50154119}.png](32B1F102-E0D4-4139-8695-050B50154119.png)

Bam got the root flag.
