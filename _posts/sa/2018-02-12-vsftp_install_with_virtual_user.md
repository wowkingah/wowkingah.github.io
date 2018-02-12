---
layout: post
title: vsftp虚拟用户认证
date: 2018-02-12
tags: vsftp
---

### 简述  
vsftp提供三种用户认证方式：  
> 1. 匿名访问：任何人无需验证口令即可登入FTP;  
> 2. 本地用户：通过FTP服务器/etc/passwd中的用户密码来访问FTP（可限制访问用户）;  
> 3. 虚拟用户：使用虚拟用户通过pam认证。  
> 

### 安装  
```bash
[root@wowking ~]# rpm -q vsftpd
未安装软件包 vsftpd
[root@wowking ~]# yum -y install vsftpd
#### 虚拟用户的宿主用户
[root@wowking ~]# groupadd -g 66 lyfftpd
[root@wowking ~]# useradd lyfftpd -d /data/ftpdata -u 66 -g 66 -s /sbin/nologin
[root@wowking ~]# mkdir /data/ftpdata
[root@wowking ~]# mkdir /data/logs/vsftpd
[root@wowking ~]# chown lyfftpd:lyfftpd -R /data/ftpdata/
[root@wowking ~]# chown lyfftpd:lyfftpd -R /data/logs/vsftpd/
[root@wowking ~]# cat /etc/vsftpd/vsftpd.conf |grep -vE '^#|^$'
anonymous_enable=NO    # 不允许匿名访问
local_enable=YES    # 允许使用本地帐户进行FTP用户登录验证
write_enable=NO    # 不允许本地用户对FTP服务器文件具有写权限
local_umask=022
anon_upload_enable=NO    # 不允许匿名用户上传文件
anon_mkdir_write_enable=NO    # 不允许匿名用户创建目录
dirmessage_enable=YES    # 设置欢迎语
xferlog_enable=YES    # 启用上传/下载日志记录
chown_uploads=YES    # 设置是否改变匿名用户上传文件（非目录）的属主
xferlog_file=/data/logs/vsftpd/vsftpd.log    # 设置日志文件名和路径
xferlog_std_format=YES    # 如果启用，则日志文件将会写成xferlog的标准格式
idle_session_timeout=300    # 设置300s不对FTP服务器进行任何操作，则断开该FTP连接
data_connection_timeout=120    # 设置建立FTP数据连接的超时时间
async_abor_enable=NO    # 如果 FTP client 会下达“async ABOR”这个指令时，这个设定才需要启用，而一般此设定并不安全，所以通常将其取消
ascii_upload_enable=YES    # 启用ASCII 模式上传
ascii_download_enable=YES    # 启用ASCII 模式下载
ftpd_banner=Welcome to LYF FTP service.    # 定义欢迎话语的字符串
chroot_local_user=YES    # 当chroot_list_enable=NO，chroot_local_user=YES，allow_writeable_chroot=YES时，所有的用户均不能切换到其他目录
chroot_list_enable=NO
allow_writeable_chroot=YES
listen=NO
listen_ipv6=YES
userlist_enable=NO    # 不启用user_list文件
tcp_wrappers=YES    # 设置vsftpd与tcp wrapper相结合来进行主机的访问控制，vsftpd服务器会检查/etc/hosts.allow 和/etc/hosts.deny 中的设置来决定请求连接的主机，是否允许访问该FTP服务器
use_localtime=YES
pam_service_name=vsftpd    # 设置PAM使用的名称
guest_enable=YES    # 启用虚拟用户
guest_username=lyfftpd    # 映射虚拟用户，宿主用户
user_config_dir=/etc/vsftpd/vuser_conf    # 定义用户配置文件
virtual_use_local_privs=NO    # YES时，虚拟用户使用与本地用户相同的权限。NO时，虚拟用户使用与匿名用户相同的权限
pasv_enable=YES
pasv_min_port=10060    # 在PASV工作模式下，数据连接可以使用的端口范围的最小端口，0 表示任意端口
pasv_max_port=10090
accept_timeout=5    # 设置建立FTP连接的超时时间为5s
anon_world_readable_only=NO    # 不允许匿名登入者下载
anon_other_write_enable=NO    # 不允许匿名登入者更多于上传或者建立目录之外的权限
#### 创建虚拟用户，基数行为用户名，偶数行为密码。db_load生成密码文件后，该文件可删除
[root@wowking ~]# mkdir /etc/vsftpd/vuser_conf
[root@wowking ~]# cat > /etc/vsftpd/vuser_passwd
wowking
123456
[root@wowking ~]# db_load -T -t hash -f /etc/vsftpd/vuser_passwd /etc/vsftpd/vuser_conf/vuser_passwd.db
[root@wowking ~]# chmod 600 /etc/vsftpd/vuser_passwd.db
[root@wowking ~]# rm /etc/vsftpd/vuser_passwd
#### 指定虚拟用户数据目录（根目录）
[root@wowking ~]# mkdir /data/ftpdata/wowking
[root@wowking ~]# chown lyfftpd:lyfftpd -R /data/ftpdata/wowking
#### 指定虚拟用户配置文件
[root@wowking ~]# cat > /etc/vsftpd/vuser_conf/wowking
local_root=/data/ftpdata/wowking
virtual_use_local_privs=YES
write_enable=YES
[root@wowking ~]# cat /etc/pam.d/vsftpd
##%PAM-1.0
auth       required    pam_userdb.so    db=/etc/vsftpd/vuser_conf/vuser_passwd
account    required    pam_userdb.so    db=/etc/vsftpd/vuser_conf/vuser_passwd
[root@wowking ~]# systemctl status vsftpd.service
vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled)
   Active: inactive (dead)

[root@wowking ~]# systemctl start vsftpd.service
[root@wowking ~]# systemctl enable vsftpd.service
ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service'
```

### 开放firewall  
```bash
[root@wowking ~]# firewall-cmd --get-services
RH-Satellite-6 amanda-client bacula bacula-client dhcp dhcpv6 dhcpv6-client dns ftp high-availability http https imaps ipp ipp-client ipsec jenkins_8080 kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3s postgresql proxy-dhcp radius rpc-bind samba samba-client smtp ssh telnet tftp tftp-client transmission-client vnc-server wbem-https
[root@wowking ~]# firewall-cmd --permanent --add-service=ftp
success
[root@wowking ~]# systemctl reload firewalld.service
[root@wowking ~]# firewall-cmd --list-services
ftp ssh
```

### 测试  
```bash
[root@wowking ~]# yum install lftp
[root@wowking ~]# lftp -u wowking,123456 127.0.0.1
```


#### 添加虚拟用户
```bash
[root@wowking ~]# cat > /etc/vsftpd/vuser_conf/t1
local_root=/data/ftpdata/t1
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=NO
[root@wowking ~]# mkdir /data/ftpdata/t1
[root@wowking ~]# cat > /etc/vsftpd/vuser_passwd
t1
111111
[root@wowking ~]# db_load -T -t hash -f /etc/vsftpd/vuser_passwd /etc/vsftpd/vuser_conf/vuser_passwd.db
```

#### 仅下载权限   
```bash
local_root=/data/ftpdata/wowking
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=NO
```

#### 上传、下载、创建目录权限  
```bash
local_root=/data/ftpdata/wowking
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```

#### 所有权限  
```bash
[root@wowking ~]# cat /etc/vsftpd/vuser_conf/wowking
local_root=/data/ftpdata/wowking
virtual_use_local_privs=YES
write_enable=YES
```

#### 虚拟用户高级参数  
> 当virtual_use_local_privs=YES：虚拟用户和本地用户有相同的权限；  
> 当virtual_use_local_privs=NO：虚拟用户和匿名用户有相同的权限，默认是NO；  
> 当virtual_use_local_privs=YES，write_enable=YES：虚拟用户具有写权限（上传、下载、删除、重命名）；  
> 当virtual_use_local_privs=NO，write_enable=YES，anon_world_readable_only=YES，
anon_upload_enable=YES：虚拟用户不能浏览目录，只能上传文件，无其他权限；  
> 当virtual_use_local_privs=NO，write_enable=YES，anon_world_readable_only=NO，
anon_upload_enable=NO：虚拟用户只能下载文件，无其他权限；  
> 当virtual_use_local_privs=NO，write_enable=YES，anon_world_readable_only=NO，
anon_upload_enable=YES：虚拟用户只能上传和下载文件，无其他权限；  
> 当virtual_use_local_privs=NO，write_enable=YES，anon_world_readable_only=NO，
anon_mkdir_write_enable=YES：虚拟用户只能下载文件和创建文件夹，无其他权限；  
> 当virtual_use_local_privs=NO，write_enable=YES，anon_world_readable_only=NO，
anon_other_write_enable=YES：虚拟用户只能下载、删除和重命名文件，无其他权限；  

### reference  
http://blog.csdn.net/hanchao_h/article/details/72731996?locationNum=11&fps=1