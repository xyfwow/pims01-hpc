# 集群远程启动

## IPMI启动主节点和磁盘阵列

首先通过控制机启动主节点

```
ipmitool -I lanplus -H 192.168.21.211 -U ADMIN -P ADMIN power on
```

等主节点开机后，通过IPMI启动三个磁盘阵列

```
ipmitool -I lanplus -H 10.10.10.11 -U ADMIN -P ADMIN power on
ipmitool -I lanplus -H 10.10.10.12 -U ADMIN -P ADMIN power on
ipmitool -I lanplus -H 10.10.10.13 -U ADMIN -P ADMIN power on
```

磁盘阵列启动完毕，手动挂载磁盘阵列，命令位于主节点`/etc/rc.local`中

```
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
/usr/sbin/iptables-restore /opt/etc/iptables.rules
service openibd restart
service opensmd restart
ifup bond0
ifconfig -s bond0 mtu 65520
sleep 10
modprobe lustre
mount.lustre idisk1@tcp0:/lufs /home-lustre -o localflock
/opt/SIMULIA/License/lmgrd -c /opt/SIMULIA/License/ABAQUS.lic -l /opt/SIMULIA/License/ABAQUS.log
/usr/sbin/rinetd > /dev/null
```

可以通过

```
lfs check servers
```

检查哪台没正常启动。

