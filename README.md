## Flanders writeup 
by **Staff Member ~ 0xRar** \
https://echoctf.red


### Scanning :
```
nmap -T4 -A -Pn -p- 10.0.100.34


Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-10 20:10 EST
Nmap scan report for 10.0.100.34
Host is up (0.12s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
6022/tcp open  ssh     libssh 0.8.1 (protocol 2.0)
| ssh-hostkey: 
|_  2048 9c:42:2e:fa:60:30:95:dd:a0:60:80:1f:fd:ae:77:86 (RSA)
```

### Gain Access :
As we see in the nmap scan we have an ssh open on port `6022` the first step is to search for default creds for `libssh`

```
creds : {username = my**** || password = mypass****}
```

lets connect to ssh: `ssh -p 6022 myuser@10.0.100.34`

connected to ssh as a normal user `ETSCTF` 

flags: /etc/passwd, /etc/shadow, environment(proc/1/environ), root 

the only flag you can get now is `/etc/passwd` its the only flag we can access with a user access

### Privilege Escalation : 
* Lets use the command `ss -ant` to see if there is any services. Running locally, we have another ssh port open on `127.0.0.1:22`. 
* in `/home/ETSCTF` if we use `ls -la` we can see hidden files and dir's. there is a dir that cought my eye which is `.ssh` this dir usually used to store
information about ssh or ssh keys lets try logging in with the key:

```sh
ssh -p 22 -i mykey root@127.0.0.1
```

Oops! we need a passphrase lets get back and read the target's description 

```
His favorite catch phrase is Okily Dokily
```

lets try out `Okily Dokily`, no luck. Try without spaces, BOOM! we have root access to the target flanders

go get your flags GOOD LUCK :)


* For any questions feel free to join the echoCTF.RED discord server: https://discord.gg/2SRkBHHGQB

_Special thanks to databus(Pantelis Roditis) for providing these amazing machines<br> and the development of the platform._

### Disclaimer :
This writeup is just a shortcut the actual machine took further time and research.
