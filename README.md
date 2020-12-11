# echoCTF flanders / 10.0.100.34 Writeup
by **Staff Member ~ 0xRar** https://echoctf.red/profile/2163092
![](https://echoctf.red/profile/2163092/badge)

## Target details
**Description**
> Flanders simple and kind, always ready to give a helping hand. His favorite catch phrase is `Okily Dokily`
>
> _Catch phrase sounds like a pass phrase, only without space_

* https://echoctf.red/target/13
* Beginner, Rootable, Timed
* 4: Flags (root, env, 2:system)
* 1: Service
* 4,800 pts

## Scanning
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

## Gaining Access
After looking around for `libssh`, I found two different potential ways to try out.
* One was to try for default username and passwords
* The other was to exploit a vulnerability on that particular version of libssh (CVE-2018-10933)

After a quick check to see if there are any default credentials for this service, proved fruitful. I managed to find these https://github.com/simonsj/libssh/blob/master/examples/ssh_server_fork.c#L51-L52

Giving them a try with `ssh -p 6022 myuser@10.0.100.34` and... it worked, i was logged in as the user `ETSCTF`.

The only flag i could grab at this stage was `/etc/passwd` so I did just that and claimed the flag from the file.

## Privilege Escalation
Now we need to escalate our privileges... Lets gather a few extra details about the system and figure out how to escalate into `root`.

1. Checking for local files i found that there is a file at `.ssh/mykey` and by the looks of it, it seems it is an ssh private key
```sh
ETSCTF@flanders:~$ ls .ssh/
mykey  mykey.pub
```

2. Checking for other listening services on the system with `ss -ant` revealed another ssh service running but its listening
```sh
ETSCTF@flanders:/$ ss -ant
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      10           *:6022                     *:*
LISTEN     0      128    127.0.0.1:22                     *:*
```

3. Checking for running processes provided a few more details, such as that there are two ssh servers running on the system. The one we are connected on runs as user `ETSCTF` which explains why we got droped to that user, when we logged in. The other however is running as root.
```sh
ETSCTF@flanders:~$ ps -eafww
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 14:11 pts/0    00:00:00 /bin/bash /entrypoint.sh default
root        20     1  0 14:11 ?        00:00:00 /usr/sbin/sshd
ETSCTF      39    38  0 14:11 ?        00:00:00 /usr/src/build/examples/ssh_server_fork --hostkey=/etc/ssh/Essh_host_rsa_key --ecdsakey=/etc/ssh/Essh_host_ecdsa_key --dsakey=/etc/ssh/Essh_host_dsa_key --rsakey=/etc/ssh/Essh_host_rsa_key -p 6022 0.0.0.0
```

Combining the details from all the previous steps, I tried to connect to the local service with the key I found,
```sh
ETSCTF@flanders:~$ ssh -p 22 -i ~/.ssh/mykey root@127.0.0.1
```

Unfortunately the key seems to be protected by a passphrase. After a few random attempts i ended up re-reading the target description again, and this time certain things started to sound a bit more revealing: **His favorite catch phrase is `Okily Dokily`**

The first attempt, using `Okily Dokily`, was not successful. Tried again without spaces (`OkilyDokily`) and ... **BOOM!** we have `root` access on flanders.


Now go get your flags and **GOOD LUCK!** :)


If you have any questions, i can be reached at the [echoCTF discord server](https://discord.gg/2SRkBHHGQB)

_Special thanks to Echothrust's `databus (Pantelis Roditis)` for providing these amazing machines \
and for developing the platform._

## Disclaimer
This writeup is just a quick step-by-step of the target solution, the actual machine took a lot more time and research.
