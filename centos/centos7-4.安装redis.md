# centos7-安装redis

```sh
#下载编译安装
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
tar -xzf redis-4.0.9.tar.gz -C /usr/local/
rm -rf redis-4.0.9.tar.gz
ln -s /usr/local/redis-4.0.9 /usr/local/redis
cd /usr/local/redis
make && make install
cd -

useradd -s /sbin/nologin -M redis
chown -R redis.redis /usr/local/redis
chown -R redis.redis /usr/local/redis-4.0.9

#配置服务文件
echo '
[Unit]
Description=The redis-server Process Manager
After=syslog.target network.target

[Service]
User=redis
Group=redis
Type=simple
PIDFile=/var/run/redis_server.pid
ExecStart=/usr/local/redis/src/redis-server --port 5555 --protected-mode no --requirepass "123456" --notify-keyspace-events "Egx"
ExecReload=/bin/kill -USR2 $MAINPID
ExecStop=/bin/kill -SIGINT $MAINPID

[Install]
WantedBy=multi-user.target
' > /lib/systemd/system/redis-server.service
#--port 设置端口
#--protected-mode 非保护模式才能远程连接
#--requirepass 需要密码
#--notify-keyspace-events 设置Egx，spring session需要

#启动服务
systemctl daemon-reload
systemctl start redis-server
systemctl enable redis-server
systemctl status redis-server

netstat -lntp

#客户端登录
#/usr/local/redis/src/redis-cli -p 5555 -a '123456'

```