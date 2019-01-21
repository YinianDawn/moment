# centos7-1.初始化

```sh
#hostnamectl set-hostname alpha
#hostname

#更改端口
echo 'Port 2222' >> /etc/ssh/sshd_config
#安装依赖
yum -y install bzip2-devel curl-devel db4-devel libjpeg-devel libpng-devel freetype-devel
yum -y install libXpm-devel gmp-devel libc-client-devel openldap-devel unixODBC-devel 
yum -y install postgresql-devel sqlite-devel aspell-devel net-snmp-devel libxslt-devel 
yum -y install libxml2-devel pcre-devel mysql-devel pspell-devel 
yum- y install libmemcached libmemcached-devel zlib-devel 
yum -y install xinetd telnet telnet-server gcc cmake vim wget lrzsz tree zip unzip
yum -y update
#安装自动更新
yum -y install yum-cron
yum -y install cronie
yum -y install yum-cron
sed -i "s|apply_updates = no|apply_updates = yes|g" /etc/yum/yum-cron.conf
systemctl start crond
systemctl start yum-cron
```

