

# Ansible
Ansible is Configuration Management Tool, is an open-source automation engine that automates cloud provisioning, configuration management, and application deployment. Once installed on a control node, Ansible, which is an agentless architecture, connects to a managed node via SSH.


Ansible Installation information can aslo be found [here]( 
http://docs.ansible.com/ansible/intro_installation.html)

***Environment***     
The following environment variables may specified.   
ANSIBLE_HOSTS - Override the default ansible hosts file   
ANSIBLE_LIBRARY - Override the default ansible module library path    

***Files***        
/etc/ansible/hosts - Default inventory file .   
/usr/share/ansible/ - Default module library .   
/etc/ansible/ansible.cfg - Config file, used if present .  
~/.ansible.cfg - User config file, overrides the default config if present .   


## Ansible Installation
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

Click **[here]**(https://github.com/shakeer-linux/ansible/blob/master/passwordless-ssh.md) to set up passwordless SSH authentication between Master to Slaves.

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

## AD-HOC
**Ansible Basic Installation done node And Now run some Basic AD-HOC commands and Playbooks for practices.**


ping
```
root@ansilbemaster:~# ansible -m ping webserver
192.168.11.51 | success >> {
    "changed": false,
    "ping": "pong"
}

root@ansilbemaster:~#
```

free
```
root@ansilbemaster:~# ansible -m shell -a 'free -m' webserver
192.168.11.51 | success | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:           992        502        490          0         23        405
-/+ buffers/cache:         73        919
Swap:         1019          0       1019
```
shell commands
```
root@ansilbemaster:~# ansible -m shell -a 'apt-get install -y tree' webserver
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

root@ansilbemaster:~# ansible -m shell -a 'apt-get remove -y tree' webserver
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

root@ansilbemaster:~#
```
For Gathering FACTS
```
root@ansilbemaster:~# ansible all -m setup
192.168.11.52 | FAILED => SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue
192.168.11.51 | success >> {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.11.51",
            "192.168.122.1"
        ],

...
...
...
...
...
...

        "ansible_swapfree_mb": 1019,
        "ansible_swaptotal_mb": 1019,
        "ansible_system": "Linux",
        "ansible_system_vendor": "innotek GmbH",
        "ansible_user_id": "root",
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virbr0": {
            "active": false,
            "device": "virbr0",
            "id": "8000.000000000000",
            "interfaces": [],
            "ipv4": {
                "address": "192.168.122.1",
                "netmask": "255.255.255.0",
                "network": "192.168.122.0"
            },
            "macaddress": "26:50:1c:8b:5f:19",
            "mtu": 1500,
            "promisc": false,
            "stp": true,
            "type": "bridge"
        },
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "virtualbox"
    },
    "changed": false
}

root@ansilbemaster:~#
```
## Ansible Playbook

*Running a playbook*   
**Playbook**  :  A playbook is an entirely different way of running Ansible, that is far more powerful.

What is a **play**?   
A play is a set of tasks mapped to a set of hosts. Plays are organised inside a text file called a playbook.

**Running a playbook**    
Below playbook runs one task, on our one host, host01. Note the indentation - it's important for how the file gets parsed. Blank lines are ignored, but makes the playbook more readable for humans.

```
cat site.yml

---
- hosts: webserver
  tasks:
    - name: ensure latest sysstat is installed
      apt:
        name: sysstat
        state: latest
```


**To run the playbook**, use the **ansible-playbook** command with the inventory file myhosts:
```
ansible-playbook -i myhosts site.yml
```
```
root@ansilbemaster:~/playbook# ansible-playbook site.yaml

PLAY [webserver] **************************************************************

GATHERING FACTS ***************************************************************
ok: [192.168.11.51]

TASK: [ensure latest sysstat is installed] ************************************
changed: [192.168.11.51]

PLAY RECAP ********************************************************************
192.168.11.51              : ok=2    changed=1    unreachable=0    failed=0

root@ansilbemaster:~/playbook#
```

Ansible should return the result 'Changed=1', indicating that the package was installed.

Playbook breakdown
What happened here?
```
--- denotes the beginning of a YAML file
hosts: host tells Ansible to run the tasks on the host host    

- name: is basically a comment, describing what the task does   

apt: specifies the module we want to use   

name: is an argument to the apt module, that specifies the name of the package to install.
```
```
The apt module allows you to specify the state you wish the package to be in. If you want a specific version, you append it to the package name, 

for example:
- name: ensure sysstat is installed at version 10.2.0-1
  apt:
    name: sysstat=10.2.0-1
    state: installed
```
```
If you want to ensure that the package is not installed, you can declare that with state: absent, and Ansible will ensure it.

for example:
- name: ensure sysstat is removed
  apt:
    name: sysstat
    state: absent

```

**Do Basic Tasks here for Best practices using PlayBooks. [Click here] (https://github.com/shakeer-linux/ansible/blob/master/ansible-tasks1) for tasks**






### Ansible Useful Optinons while runing ansible commands.        

***-v, --verbose***       
Verbose mode, more output from successful actions will be shown. Give up to three times for more output.   

***-i PATH, --inventory=PATH***   
The PATH to the inventory hosts file, which defaults to /etc/ansible/hosts.  

***-M DIRECTORY, --module-path=DIRECTORY***   
The DIRECTORY to load modules from. The default is /usr/share/ansible.  

***-e VARS, --extra-vars=VARS***    
Extra variables to inject into a playbook, in key=value key=value format.   

***-f NUM, --forks=NUM***     
Level of parallelism. NUM  is specified as an integer, the default is 5.   

***-k, --ask-pass***    
Prompt for the SSH password instead of assuming key-based authentication with ssh-agent.   

***-K, --ask-sudo-pass***     
Prompt for the password to use for playbook plays that request sudo access, if any.   

***-U, SUDO_USER, --sudo-user=SUDO_USER***      
Desired sudo user (default=root).   

***-T SECONDS, --timeout=SECONDS***     
Connection timeout to use when trying to talk to hosts, in SECONDS.   

***-s, --sudo***    
Force all plays to use sudo, even if not marked as such.   

***-u USERNAME, --remote-user=USERNAME***     
Use this remote user name on playbook steps that do not indicate a user name to run as.   

***-c CONNECTION, --connection=CONNECTION***     
Connection type to use. Possible options are paramiko (SSH), ssh, and local. local is mostly useful for crontab or kickstarts.   

***-l SUBSET, --limit=SUBSET***     

Further limits the selected host/group patterns.  
*https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet*
