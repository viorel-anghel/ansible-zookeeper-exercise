# Install zookeeper on CentOS 7

based on https://blog.redbranch.net/2018/04/19/zookeeper-install-on-centos-7/
with updates and corrections in 2020-05-28

- work as root
- firewall - there in no firewall on our VMs on private interface

```bash

# set those variables to match your hosts
ZK1=ansible25
ZK2=ansible26
ZK3=ansible27

# from here you can just copy-paste everything

yum install -y java-1.8.0-openjdk
groupadd zookeeper 
useradd -g zookeeper -s /sbin/nologin zookeeper
# do not use /opt/zookeeper as user home
yum install -y wget

cd /opt 
wget http://mirrors.ukfast.co.uk/sites/ftp.apache.org/zookeeper/zookeeper-3.5.9/apache-zookeeper-3.5.9-bin.tar.gz
tar xzvf apache-zookeeper-3.5.9-bin.tar.gz 
ln -s apache-zookeeper-3.5.9-bin zookeeper
mkdir /opt/zookeeper/data

# make sure you have the correct hostnames below
cat <<EOF >/opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/zookeeper/data
clientPort=2181
server.1=${ZK1}:2888:3888
server.2=${ZK2}:2888:3888
server.3=${ZK3}:2888:3888
EOF

echo 1 >/opt/zookeeper/data/myid
# please note this file will contain 2 and 3 on the other hosts

chown -R zookeeper:zookeeper /opt/zookeeper/

# do not start zookeeper now with /opt/zookeeper/bin/zkServer.sh start
# because it will create files as root under /opt/zookeeper/data

cat <<EOF >/usr/lib/systemd/system/zookeeper.service
[Unit]
Description=Zookeeper Service

[Service]
Type=simple
WorkingDirectory=/opt/zookeeper/
PIDFile=/opt/zookeeper/data/zookeeper_server.pid
SyslogIdentifier=zookeeper
User=zookeeper
Group=zookeeper
ExecStart=/opt/zookeeper/bin/zkServer.sh start
ExecStop=/opt/zookeeper/bin/zkServer.sh stop
Restart=always
TimeoutSec=20
SuccessExitStatus=130 143
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable zookeeper
systemctl start zookeeper

# don't worry if it will exit
# that's because there is no quorum

```
 


