## Flanders writeup 
by **Staff Member ~ 0xRar** \
https://echoctf.red


### Scanning
First thing to do is run nmap and see what kind of services are running on the system

```sh
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

The only service that is running is this ssh service which is based on `libssh 0.8.1`.

### Gaining Access
After looking around for `libssh`, I found two different potential ways to try out.
* One was to try for default username and passwords
* The other was to exploit a vulnerability on that particular version of libssh (CVE-2018-10933)

After a quick check to see if there are any default credentials for this service, proved fruitful. I managed to find these https://github.com/simonsj/libssh/blob/master/examples/ssh_server_fork.c#L51-L52

Giving them a try with `ssh -p 6022 myuser@10.0.100.34` and... it worked, i was logged in as the user `ETSCTF`.

The only flag i could grab at this stage was `/etc/passwd` so I did just that and claimed the flag from the file.

### Privilege Escalation
Now we need to escalate our privileges... Lets gather a few extra details about the system and figure out how to escalate into `root`.

1. Checking for local files i found that there is a file at `.ssh/mykey` and by the looks of it, it seems it is an ssh private key
2. Checking for other listening services on the system with `ss -ant` revealed another ssh service running but its listening on `localhost:22`

Combining the two, I tried to connect to the local service with the key i found,
```sh
ssh -p 22 -i ~/.ssh/mykey root@127.0.0.1
```

Unfortunately the key seems to be protected by a passphrase. After a few random attempts i ended up re-reading the target description again, and this time certain things started to sound a bit more revealing: **His favorite catch phrase is `Okily Dokily`**

The first attempt, using `Okily Dokily`, was not successful. Tried again without spaces (`OkilyDokily`) and ... **BOOM!** we have `root` access on flanders.


Now go get your flags and **GOOD LUCK!** :)


If you have any questions, i can be reached at the [echoCTF discord server](https://discord.gg/2SRkBHHGQB)

_Special thanks to Echothrust's `databus (Pantelis Roditis)` for providing these amazing machines \
and for developing the platform._

### Disclaimer :
This writeup is just a quick step-by-step of the target solution, the actual machine took a lot more time and research.
