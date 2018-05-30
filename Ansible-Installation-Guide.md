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
192.168.11.50	AnsilbeMaster
192.168.11.51	AnsilbeSlave

root@ansibleslave:~# cat /etc/hosts
127.0.0.1	localhost
192.168.11.50   AnsilbeMaster
192.168.11.51   AnsilbeSlave
```
On Both master and slave install below packages.

```
apt-get update
apt-get install software-properties-common
apt-get install ansible
```
