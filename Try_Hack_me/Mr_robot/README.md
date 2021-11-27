# **TryHackMe**
## Mr_Robot
![intro](https://user-images.githubusercontent.com/70982180/143719819-d899c4e7-96e0-47ff-b830-d7efccf64b0e.png)

# Intro
The scope of this engagement will be testing the security asset of this server , also finding the 3 flags .

# 0x00 Reconnaissance
We will start with portscan with nmap top 1000 ports  with service enumaration :

![image](https://user-images.githubusercontent.com/70982180/143719844-0734bc16-3b74-4104-8877-56786e4a52ad.png)

We will continue with a full port scan :

![image](https://user-images.githubusercontent.com/70982180/143719862-97ee0eac-eca1-4911-b9d7-bfd8229abdc9.png)

The server is exposing 3 ports  ,  our attacking surface will be :
⦁	Port 22 ssh  - closed ? 
⦁	Port 80 apache httpd
⦁	Port 443 ssl apache httpd

Initial scan shows that we potentially have 2 vector of attack 
http server and ssh .

Let’s start with enumerating port 80 and port 443 , 
since SSH port 22 is pretty much always secured.

# 0x01 Enumeration

Visiting the url:http://10.10.180.19 we are presented with an interactive html page : 

![image](https://user-images.githubusercontent.com/70982180/143719904-36600003-2617-48be-ae9e-77a5e0ebc28d.png)

Let’s check the source page :

![image](https://user-images.githubusercontent.com/70982180/143719943-d400d6c0-c019-4e0d-a231-2b25cc2d7c4f.png)

A little easterEGG we found but for the rest nothing interesting , 
let’s check for robots.txt files :

![image](https://user-images.githubusercontent.com/70982180/143719956-2d442da6-9a1e-4d24-9d93-abd4e7a356ed.png)

The server is exposing a robots.txt file with some interesting entries ,
Let's navigate to those entry and see what we have :

![image](https://user-images.githubusercontent.com/70982180/143720025-f92c0dc5-f988-42fe-840c-2c8829ef15c9.png)

![image](https://user-images.githubusercontent.com/70982180/143720039-1059f67c-fd8e-48a3-9dc4-8121cfc02a50.png)

As we can see we found the first flag , also we have found a file name 
fsocity.dic (misspelled for Fsociety )  containing a list of name :

![image](https://user-images.githubusercontent.com/70982180/143720054-707296b7-a7d7-43d1-91a5-3c5ab9fb5100.png)

Ok let’s continue our enumeration with gobuster 
A directory bruteforcing tool :

![image](https://user-images.githubusercontent.com/70982180/143720070-f1000688-8ebe-468e-a927-269e88bc0f9a.png)

Thanks to this report I can assume that the site is running WordPress

```
/wp-login.php         (Status: 200) [Size: 2606] 
/wp-register.php

```

Also is running some version of PHP 

Let’s investigate more about wp-login.php which is the user login :

![image](https://user-images.githubusercontent.com/70982180/143720136-263131cc-3c5f-4812-86bc-63c15e0e1907.png)

We will try to enumerate users from this login page, 
we already a list so we are gonna use Burp in order to brute force the login page :

![image](https://user-images.githubusercontent.com/70982180/143720145-a0494bbe-c8c7-4d48-907c-33ab3ffeeb2f.png)

We can see that only when providing the username Elliot 
we ha different respons lenght Let’s verified this behavior 
trying to log in with the user Elliot 

![image](https://user-images.githubusercontent.com/70982180/143720179-ee3b927a-eb78-4049-b51d-325ad14fbbdf.png)

Infact we can see the different error message 
that means that the account elliot exist 
but we dont have a password yet . 
Let’s try to bruteforce the password with the same list, since this is a wordpress site ,  this time I will use a tool called WPSCAN , that has builtin feature for brute forcing :

![image](https://user-images.githubusercontent.com/70982180/143720213-66b3bd4b-d9db-46d0-8b16-1217618f563d.png)


We were able to find credentials for the WordPress pannel :

## **Elliot:ER28-0652**

![image](https://user-images.githubusercontent.com/70982180/143720294-1528b294-b6a7-4d49-9827-61ae3bfd1df9.png)

# **0x02 Exploiting**

We successfully logged in as a user , 
As we can see we can enumerate the CMS version 

Wordpress 4.3.1 

Normally I will look at the plugin option , to load a new plugin 
or to modify one that is already installed , 
Since the site runs PHP , I will try to inject some malicious 
PHP code to see if the server will run it:

In the section “Appearance” -> Editor   we can modify an already installed plug in and  upload it , the code we will add is a PHP reverse shell 
 
![image](https://user-images.githubusercontent.com/70982180/143720395-bc9dc721-3ef7-45ac-be3d-7ad8526e863a.png)


 ![image](https://user-images.githubusercontent.com/70982180/143720446-4f63a200-960e-4d0d-a6bf-35e7a26e280c.png)

#***0x03 Privilege Escalation*

We got a reverse shell with the user daemon ,
First thing to do is checking home directory to see which user are on the machine , We have a user “robot” , It’s home directory is accessible 

![image](https://user-images.githubusercontent.com/70982180/143720467-24ab78f1-4576-44c5-8762-ea1c1e4eedca.png)

As we can see , the key is only readable with the username robot , so we need to find a way to access the user robot first in order to read the flag , we have an hash and we can read the contenct of the file 

![image](https://user-images.githubusercontent.com/70982180/143720483-0ba1dd6b-5bd0-4d87-b7d4-737b4052ec51.png)

With the help of “John” we will  try to crack the hash with the wordlist  “Rockyou"

![image](https://user-images.githubusercontent.com/70982180/143720497-31842d24-f69d-45fe-8052-bbdd7a9eb6a3.png)

Password found for the user robot 

## **robot: abcdefghijklmnopqrstuvwxyz**

Ok now in order to login with the user robot we need to upgrade our shell to a fully interactive shell , to do sowe will need python to import a fully interactive TTY shell , so we can use the command “su”
to log in with the user robot :

![image](https://user-images.githubusercontent.com/70982180/143720578-0c59687d-1bb6-4101-abfb-09114b741c04.png)

Now we have uparrow and tab completation , 
let’s  log in as the new user : 

![image](https://user-images.githubusercontent.com/70982180/143720585-798508cd-1f3a-461b-978e-0637890f8606.png)

We got the second key : 

![image](https://user-images.githubusercontent.com/70982180/143720594-98416ccc-7448-48f5-b41e-0166fcbc1242.png)

Robot can not run any executable as root unfortunately , 
lets continue the enumaration , I will host from my kali machine a privilege execution sh script called “Linpeas” , to do so  I will need to set a python http server , and from the target machine I will wget it and run it in /tmp/directory:

![image](https://user-images.githubusercontent.com/70982180/143720610-9696fc8e-69d9-498b-a2a8-7288d4aa3335.png)

![image](https://user-images.githubusercontent.com/70982180/143720638-7c5f42bc-f247-48b6-abec-809d721f73f7.png)

From the output of the program we were able to find that nmap is running with sudo privileges , therefore we can launch nmap with the flag “--interactive” , we are presented with a command line where we can execute command , therefore  we can execute a shell command . In this case nmap will not drop privileges , 
so we are executing a shell command with root privileges :

![image](https://user-images.githubusercontent.com/70982180/143720647-f8e97ce4-5881-432d-bc9c-c7819b8175b0.png)

We have successfully got root privileges , the last flag will be in the root directory :


![image](https://user-images.githubusercontent.com/70982180/143720659-038f8480-2337-4b6f-a36d-154fb31996fc.png)


## **0x04 Conclusions** 

With the help of an internal we were able to find a dictionary
exposed on the web server , with the help of this dictionary we could enumerate users at the admin login page of the CMS .
Once we found the right useraname we conduct a  password brute-force attack against the admin pannel , after the login-in we were able to find a code injection  vulnerability in the plug-in  section in which was possibile to run arbitrary commands on the server .
 Once discovered this vulnerabilty we were able to get a reverse shell with low privilege user (www-data) , after some information gathering conduct on the machine , we were able to find some hashed credentials stored in the user home directory , once cracked and logged in with the user , with an automated privilege escalation tool we were able to discover a suid binary from which we could escalade our privilges to root.

## **0x05 Lession Learned**

⦁	 Our foothold was gained thanks to an internal  , this can not be remadiated so easily ,  however  the Wordpress  version was outdate , so remeber to always update at the most recent version your software 

⦁	Do not store password or password hashs in world readable folders 

⦁	Limit the use of SUID on dangerous binaries 

