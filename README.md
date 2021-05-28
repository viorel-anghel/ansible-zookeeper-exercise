# Ansible zookeeper exercise

## What do we want to do

Install a zookeeper cluster with 3 nodes, all VMs with CentOS 7. 
Please note we don not have official packages for zookeeper (like in Debian/Ubuntu).

Manual installation procedure is in this directory: `install-procedure.md`

## Setup assumptions

We work on a VM which has Ansible installed. This is our master machine. 

From this one we can ssh as root with key (no passwords involved) to another 3 
VMs, all with OS CentOS version 7.

All machines are using the same /etc/hosts file to enable the use of short 
hostnames instead of IPs. Something like:

``` /etc/hosts file
127.0.0.1      localhost
10.135.188.4   ansible00
10.135.187.247 ansible27
10.135.187.238 ansible26
10.135.187.237 ansible25
``` 

## How you are supposed to work on this exercise
1. Use the steps from ```install-procedure``` and do a manual installation 
on the first VM. 
1. Try to convert those steps into a playbook.
1. Convert the playbook into a role.

## In which order to read files in here
- start with this one README.md (obviously ;-)
- install-procedure.md
- ansible/ansible.cfg
- ansible/hosts
- ansible/files/*
- ansible/zookeeper.yml
- ansible/zookeeper-role.yml
- ansible/roles/zookeeper/*

### Checking the final result
1. systemctl status zookeeper should show ok on all hosts
2. running those commands one of the hosts you should see one of them 
```Mode: leader``` and the others ```Mode: follower```.
```bash
echo srvr | nc ansible25 2181
echo srvr | nc ansible26 2181
echo srvr | nc ansible27 2181
```

Example output:
```bash
[root@ansible25 ~]# echo srvr | nc ansible27 2181
Zookeeper version: 3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 19:49 GMT
Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 5
Proposal sizes last/min/max: -1/-1/-1
```

### Explanations about the playbook
1. in inventory `hosts` we define the group `zookeeper` and a 
variable `zookeeper_id` per host
2. the playbook `zookeeper.yml` is straightforward and has some comments inside
3. there are two templates zoookeeper configuration files: `files/myid.j2` 
and `files/zoo.cfg.j2`
4. `ansible-playbook zookeeper.yml` 

### Converting to a role
1. check everything under `roles/zookeeper`. mostly copy paste from the playbook 
and split into specific files and locations
2. now you see the reason for that fancy syntax from `zoo.cfg.j2`. We do not 
hardcode hostnames inside roles. Roles should be reusable with no changes
3. to run it, `ansible-playbook zookeeper-role.yml`




