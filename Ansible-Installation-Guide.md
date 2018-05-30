**Ansible**     
Ansible is Configuration Management Tool, is an open-source automation engine that automates cloud provisioning, configuration management, and application deployment. Once installed on a control node, Ansible, which is an agentless architecture, connects to a managed node via SSH.


Ansible Installation information can aslo be found [here]( 
http://docs.ansible.com/ansible/intro_installation.html)



**Please follow below procedure for ansible installation and some tasks**


Take Two Ubuntu 14 VM’s.
```sh
1. AnsibleMaster  --  IP 192.168.11.50
2. AnsibleSlave   --  IP 192.168.11.51
```

Make sure SSH package installed on both virtual machines or else install it.

```sh
root@ansilbemaster:~# cat /etc/hosts
127.0.0.1	localhost
192.168.11.50	AnsilbeMaster master
192.168.11.51	AnsilbeSlave  slave

root@ansibleslave:~# cat /etc/hosts
127.0.0.1	localhost
192.168.11.50   AnsilbeMaster master
192.168.11.51   AnsilbeSlave  slave
```
On Both master and slave install below packages.

```
apt-get update
apt-get install software-properties-common
apt-get install ansible
```
**Setup SSH Keys**  
Ansible primarily communicates with client nodes through SSH. While it certainly has the ability to handle password-based SSH authentication, SSH keys help keep things simple.

We can set up SSH keys in two different ways depending on whether you already have a key you want to use.

Create a New SSH Key Pair:

```sh
root@ansilbemaster:~# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
f0:0a:2f:18:81:06:02:b8:eb:83:8a:f3:28:9e:1a:c6 root@ansilbemaster
The key's randomart image is:
+--[ RSA 2048]----+
|*                |
|+.               |
|.o.   .          |
|o  .   o         |
| .. .   S        |
|o  o o .         |
|+E. . o          |
|*=.  .           |
|X=o              |
+-----------------+

root@ansilbemaster:~#
```

**1st Method:**  
Execute below command. this will copy the Master node SSH Public key will copied into Slave/Client node(in .ssh/authorized_keys)
```sh
root@ansilbemaster:~# ssh-copy-id root@192.168.11.51
The authenticity of host '192.168.11.51 (192.168.11.51)' can't be established.
ECDSA key fingerprint is c6:d8:17:e2:c7:7a:6a:2a:00:05:42:50:9f:75:9a:a5.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.11.51's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.11.51'"
and check to make sure that only the key(s) you wanted were added.
```
**(OR)**


**2nd Method:**  
Copy SSH Public key of master node(.ssh/id_rsa.pub) manually into the Slave/Client node file (in .ssh/authorized_keys location)

First copy the .ssh/id_rsa.pub file with scp command into remote/slave node in tmp directory. 
```sh
root@ansilbemaster:~# scp .ssh/id_rsa.pub root@192.168.11.51:/tmp
root@192.168.11.51's password:
id_rsa.pub                                                    100%  400     0.4KB/s   00:00
root@ansilbemaster:
```
Then Go to the Salve node and Copy the id_rsa.pub file content from tmp directory to the .ssh/authorized_keys file.

```sh
root@ansibleslave:~# cat /tmp/id_rsa.pub >> .ssh/authorized_keys_test
root@ansibleslave:~# cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDrG7JQD2z8N90FRJ4+G40WllKc53nSPP9zua854u41tkvZnXToN24UOLOuQFbMNPtR/k+175b663qT+XxuyYbsmcMidMW46Twb/y3QaudxigryqkKzs9fDVflETg0vnKxQs4QRyaEMXmRv8pnXODIOSX8A/4m6CTn0lnyIT6h8usCQHSHIFlm4cpQ1kjYGcMiHKA9Fr33E+/vzUdJsBS4iZoGGYhWBSPsGVO8q8cP7nDcxfuP6o2VKAGFXWInTSWONOjHwfqRvyskDEaV3obfkro7Pd5iDUG69SYosnWdPnlghpZA4D4f5UsRUBs+LHjKCv4ckhZaaBjjijOtTEmwx root@ansilbemaster
root@ansibleslave:~#
```

**Check/Validate** the password less authentication without giving password.
```sh
root@ansilbemaster:~# ssh 192.168.11.51
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Wed May 30 14:37:31 IST 2018

  System load:  0.0               Users logged in:       1
  Usage of /:   2.7% of 47.84GB   IP address for eth0:   10.0.2.15
  Memory usage: 8%                IP address for eth1:   192.168.11.51
  Swap usage:   0%                IP address for virbr0: 192.168.122.1
  Processes:    107

  Graph this data and manage this system at:
    https://landscape.canonical.com/

New release '16.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed May 30 11:33:45 2018 from 192.168.11.1
root@ansibleslave:~# exit
logout
Connection to 192.168.11.51 closed.
root@ansilbemaster:~#
```

**Configuring Ansible Hosts**

Ansible uses by default file ***/etc/ansible/hosts*** directory to maintain record of all servers/slave nodes. we can also change default directory/file location of as per our requirements by changing default values in **/etc/ansible/ansible.cfg** file. 
```sh

root@ansilbemaster:/etc/ansible# ls
ansible.cfg  hosts
root@ansilbemaster:/etc/ansible#

Example of ansible.cfg file

root@ansilbemaster:/etc/ansible# vim ansible.cfg
...
[defaults]

# some basic default values...

hostfile       = /etc/ansible/hosts
library        = /usr/share/ansible
remote_tmp     = $HOME/.ansible/tmp
pattern        = *
forks          = 5
poll_interval  = 15
sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
transport      = smart
remote_port    = 22
...
```
Add your server information in /etc/ansible/hosts file with tags if required like.
```sh
root@ansilbemaster:/etc/ansible# vim hosts
[all]
192.168.11.51
192.168.11.52

[webserver]
192.168.11.51

[databaseserver]
192.168.11.52
```


***Ansible Basic Installation done node And Now run some Basic AD-HOC commands and Playbooks for practices.


ping
``
root@ansilbemaster:/etc/ansible# ansible -m ping webserver
192.168.11.51 | success >> {
    "changed": false,
    "ping": "pong"
}

root@ansilbemaster:/etc/ansible#
``
free
```
root@ansilbemaster:/etc/ansible# ansible -m shell -a 'free -m' webserver
192.168.11.51 | success | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:           992        502        490          0         23        405
-/+ buffers/cache:         73        919
Swap:         1019          0       1019
```
shell commands
```
root@ansilbemaster:/etc/ansible# ansible -m shell -a 'apt-get install -y tree' webserver
192.168.11.51 | success | rc=0 >>
Reading package lists...
Building dependency tree...
Reading state information...
The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 184 not upgraded.
Need to get 37.8 kB of archives.
After this operation, 109 kB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu/ trusty/universe tree amd64 1.6.0-1 [37.8 kB]
Fetched 37.8 kB in 1s (32.4 kB/s)
Selecting previously unselected package tree.
(Reading database ... 60865 files and directories currently installed.)
Preparing to unpack .../tree_1.6.0-1_amd64.deb ...
Unpacking tree (1.6.0-1) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Setting up tree (1.6.0-1) ...

root@ansilbemaster:/etc/ansible# ansible -m shell -a 'apt-get remove -y tree' webserver
192.168.11.51 | success | rc=0 >>
Reading package lists...
Building dependency tree...
Reading state information...
The following packages will be REMOVED:
  tree
0 upgraded, 0 newly installed, 1 to remove and 184 not upgraded.
After this operation, 109 kB disk space will be freed.
(Reading database ... 60872 files and directories currently installed.)
Removing tree (1.6.0-1) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...

root@ansilbemaster:/etc/ansible#
```







