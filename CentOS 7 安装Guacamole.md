
# CentOS 7 安装Guacamole及VncServer

## 下载安装guacamole

### 配置环境

安装必要环境

```
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
yum update -y
```

安装依赖包

```
yum -y install cairo-devel libjpeg-devel libpng-devel uuid-devel  
yum -y install ffmpeg-devel  freerdp-devel pango-devel libssh2-devel 
yum -y install libtelnet-devel libvncserver-devel pulseaudio-libs-devel  
yum -y install openssl-devel libvorbis-devel libwebp-devel
yum -y install freerdp-plugins
yum -y install gcc
```

### 下载安装包

下载地址

https://guacamole.apache.org/releases/1.2.0/

下载server和client

### guacamole-server安装

```
tar zxf guacamole-server-1.2.0.tar.gz -C /opt
cd /opt/guacamole-server-1.2.0
./configure --with-init-dir=/etc/init.d
```

确认所有协议是否都支持

```shell
guacamole-server version 1.2.0
------------------------------------------------

   Library status:

     freerdp2 ............ yes
     pango ............... yes
     libavcodec .......... yes
     libavformat.......... yes
     libavutil ........... yes
     libssh2 ............. yes
     libssl .............. yes
     libswscale .......... yes
     libtelnet ........... yes
     libVNCServer ........ yes
     libvorbis ........... yes
     libpulse ............ yes
     libwebsockets ....... no
     libwebp ............. yes
     wsock32 ............. no

   Protocol support:

      Kubernetes .... no
      RDP ........... yes
      SSH ........... yes
      Telnet ........ yes
      VNC ........... yes

   Services / tools:

      guacd ...... yes
      guacenc .... yes
      guaclog .... yes

   FreeRDP plugins: /usr/lib64/freerdp2
   Init scripts: /etc/init.d
   Systemd units: no

Type "make" to compile guacamole-server.
```

确认完成后使用make安装

```
make && make install
```

启用guacd服务

```shell
# /etc/init.d/guacd start
Starting guacd: guacd[2240]: INFO:      Guacamole proxy daemon (guacd) version 1.2.0 started
SUCCESS
```

### 安装guacamole client

#### 首先安装jdk和tomcat

这里使用openjdk11

```
yum update
yum install java-11-openjdk-devel
```

安装完成后查看java版本

```
java -version
```

可以通过这个命令设置默认的java版本

```
alternatives --config java
```

设置 JAVA_HOME 环境变量

首先，定位java安装路径

```
update-alternatives --config java
```

确定java路径后，复制需要的java版本，编辑 `.bash_profile`文件

```
vim .bash_profile
```

在文件最下方，添加JAVA_HOME路径

```
JAVA_HOME=”/your/installation/path/”
```

安装tomcat

```
yum install tomcat
```

将war包部署到webapp中

```shell
# find / -name webapps
/usr/share/tomcat/webapps
find: ‘/proc/10235’: No such file or directory
find: ‘/run/user/1000/gvfs’: Permission denied
/var/lib/tomcat/webapps
# cd /usr/share/tomcat/webapps/
# cp /root/guacamole/guacamole-1.2.0.war guacamole.war
# ll
total 11964
-rw-r--r--. 1 root root 12249847 Jul  3 11:02 guacamole.war
```

建立配置文件

```
cd /usr/share/tomcat
mkdir .guacamole
cd .guacamole
vim  user-mapping.xml
vim  guacamole.properties 
```

在`guacamole.properties `中配置如下

```
guacd-hostname: localhost
guacd-port:     4822
```

在`user-mapping.xml `中配置如下

```
<user-mapping>
    <authorize
            username="admin"
            password="登录账户密码">

        <!-- First authorized connection -->
        <connection name="ssh-ame">
            <protocol>ssh</protocol>
            <param name="hostname">ip地址</param>
            <param name="port">22</param>
        </connection>

        <!-- Second authorized connection -->
        <connection name="vnc-graph">
            <protocol>vnc</protocol>
            <param name="hostname">localhost</param>
            <param name="port">5901</param>
            <param name="password">vnc密码</param>
        </connection>

        <!-- Third authorized connection -->
        <connection name="ssh-graph">
            <protocol>ssh</protocol>
            <param name="hostname">localhost</param>
            <param name="port">22</param>
        </connection>
    </authorize>
</user-mapping>
```

启用tomcat服务

```
systemctl start tomcat
```

启动成功后，访问http://localhost:8080/guacamole即可访问

开启防火墙8080端口

```
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

## 配置安装vncserver

首先安装vncserver



