# centos7-安装nginx

```sh
#安装依赖
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel
yum -y install openssl openssl-devel wget
#下载编译安装
wget -c http://nginx.org/download/nginx-1.12.2.tar.gz
tar -zxvf nginx-1.12.2.tar.gz
cd nginx-1.12.2
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make && make install
whereis nginx
cd ..
rm -rf nginx-1.12.2
rm -rf nginx-1.12.2.tar.gz

useradd -s /sbin/nologin -M nginx
chown -R nginx.nginx /usr/local/nginx

#配置服务文件
echo "
[Unit]    
Description=nginx    
After=syslog.target network.target    
     
[Service]    
Type=forking    
ExecStart=/usr/local/nginx/sbin/nginx   
ExecReload=/usr/local/nginx/sbin/nginx -s reload    
ExecStop=/usr/local/nginx/sbin/nginx -s stop  
PrivateTmp=true    
     
[Install]    
WantedBy=multi-user.target 
" >> /etc/systemd/system/nginx.service

#启动服务
systemctl daemon-reload
systemctl enable nginx
systemctl is-enabled nginx
#systemctl disable nginx
systemctl status nginx
systemctl start nginx
systemctl status nginx
#systemctl restart nginx

netstat -lntp
```

