
**Please follow below procedure for ansible installation and some tasks.**

Ansible Installation:

Take Two Ubuntu 14 VM’s.
```sh
1. AnsibleMaster  --  IP 192.168.11.50
2. AnsibleSlave   -- IP  192.168.11.51
```

Make sure SSH installed on both virtual machines.



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
