# centos7-安装mysql

``` sh
#安装依赖
yum search libaio  # search for info
yum -y install libaio # install library
yum -y install ncurses-devel libaio-devel
#新增mysql非登录用户
useradd -s /sbin/nologin -M mysql
#下载安装
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz
tar -zxvf mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz -C /usr/local
rm -rf mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz
ln -s /usr/local/mysql-5.7.21-linux-glibc2.12-x86_64 /usr/local/mysql
echo 'export PATH=$PATH:/usr/local/mysql/bin' >> /etc/profile
source /etc/profile
#将配置用到的文件目录软连接到真正存放数据的地方
#这样表面上mysql用的目录都是在mysql自己目录下，实际上数据是放在数据目录下
mkdir -p /mine/data/mysql/{data,binlogs,log,etc,run}
ln -s /mine/data/mysql/data /usr/local/mysql/data
ln -s /mine/data/mysql/binlogs /usr/local/mysql/binlogs
ln -s /mine/data/mysql/log /usr/local/mysql/log
ln -s /mine/data/mysql/etc /usr/local/mysql/etc
ln -s /mine/data/mysql/run /usr/local/mysql/run
chown -R mysql.mysql /mine/data/mysql/
chown -R mysql.mysql /usr/local/mysql/{data,binlogs,log,etc,run}
#mysql配置文件
echo "
[client]
port = 9999
socket = /usr/local/mysql/run/mysql.sock
#character-set-server = utf8mb4

#[mysql]
#default-character-set=utf8

[mysqld]
port = 9999
socket = /usr/local/mysql/run/mysql.sock
pid_file = /usr/local/mysql/run/mysql.pid
datadir = /usr/local/mysql/data
default_storage_engine = InnoDB
max_allowed_packet = 128M
max_connections = 2048
open_files_limit = 65535

skip-name-resolve
lower_case_table_names=1

character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'

#innodb_buffer_pool_size = 1024M
#innodb_log_file_size = 2048M
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 0

key_buffer_size = 64M

log-error = /usr/local/mysql/log/mysql_error.log
#log-bin = /usr/local/mysql/binlogs/mysql-bin
slow_query_log = 1
slow_query_log_file = /usr/local/mysql/log/mysql_slow_query.log
long_query_time = 5

tmp_table_size = 32M
max_heap_table_size = 32M
query_cache_type = 0
query_cache_size = 32M

server-id=1
" > /etc/my.cnf

#初始化数据目录
#稍等就可以发现/usr/local/mysql/data目录下有数据了，并且实际上这个数据是在/mine/data/mysql/data下
mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
#把随机生成的密码从日志中过滤出来
grep 'temporary password' /usr/local/mysql/log/mysql_error.log
#安装rsa
mysql_ssl_rsa_setup --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/
#配置为系统服务，这是centos7的服务，在centos6不能用的
echo "
[Unit]
Description=MySQL Server
After=syslog.target network.target 

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=/usr/local/mysql/run/mysqld.pid
# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0
# Execute pre and post scripts as root
PermissionsStartOnly=true
# Needed to create system tables
#ExecStartPre=/usr/bin/mysqld_pre_systemd
# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/run/mysqld.pid $MYSQLD_OPTS
# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql
# Sets open_files_limit
LimitNOFILE = 65535
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false

[Install]
WantedBy=multi-user.target
" >> /lib/systemd/system/mysql.service
#启动mysql服务
systemctl daemon-reload 
systemctl enable mysql
systemctl is-enabled mysql
#systemctl disable mysql
systemctl status mysql
systemctl start mysql
systemctl status mysql
#systemctl restart mysql

#查看监听的端口，mysql是启动了的
netstat -lntp

#进行安全设置，聊以自慰？其实就设置密码强度，移除匿名用户什么的，没啥意思感觉
/usr/local/mysql/bin/mysql_secure_installation

#如果不能root远程登录，有可能是mysql的root用户不允许远程，把该用户的host设置成host就可以了
# mysql -u root -p
# use mysql;
# update user set host = '%' where user ='root';
# flush privileges;
# exit
```

