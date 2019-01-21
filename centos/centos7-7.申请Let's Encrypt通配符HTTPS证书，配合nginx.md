# centos7-申请Let's Encrypt通配符HTTPS证书，配合nginx

```sh
curl https://get.acme.sh | sh
source ~/.bashrc

# 替换成从阿里云后台获取的密钥
export Ali_Key="abc"
export Ali_Secret="abc"
# 换成自己的域名
acme.sh --issue --dns dns_ali -d xxx.cn -d *.xxx.cn
#等120秒就可以了
#会提示
#Your cert is in  /root/.acme.sh/xxx.cn/xxx.cn.cer 
#Your cert key is in  /root/.acme.sh/xxx.cn/xxx.cn.key 
#The intermediate CA cert is in  /root/.acme.sh/xxx.cn/ca.cer 
#And the full chain certs is there:  /root/.acme.sh/xxx.cn/fullchain.cer 

#配置nginx https
mkdir /mine/data/https
acme.sh --installcert -d xxx.cn \
        --keypath /mine/data/https/xxx.cn.key \
        --fullchainpath /mine/data/https/xxx.cn.key.pem \
        --reloadcmd "sudo systemctl restart nginx"
        
openssl dhparam -out /mine/data/https/dhparam.pem 2048

#修改nginx配置文件
sed -i "s|#gzip  on;|#gzip  on;\n\n    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;\n    ssl_prefer_server_ciphers on;|g" /usr/local/nginx/conf/nginx.conf

#其中一个配置
echo '
server {
    listen 80;
    server_name www.xxx.cn;
    rewrite ^ https://$server_name$request_uri? permanent;
}
server {
    listen 443 ssl;
    server_name www.xxx.cn;
    charset utf-8;
    location / {
        proxy_pass http://$server_name:8888;
    }

    access_log  /var/log/nginx/www.xxx.cn_access.log;
    error_log  /var/log/nginx/www.xxx.cn_error.log;

    # letsencrypt生成的文件
    ssl_certificate /mine/data/https/xxx.cn.key.pem;
    ssl_certificate_key /mine/data/https/xxx.cn.key;

    ssl_dhparam /mine/data/https/dhparam.pem;

    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets on;
    
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    # 一般推荐使用的ssl_ciphers值: https://wiki.mozilla.org/Security/Server_Side_TLS
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128:AES256:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK';
    ssl_prefer_server_ciphers on;
}
' > /usr/local/nginx/conf/etc/www.conf

/usr/local/nginx/sbin/nginx -t
systemctl restart nginx

#测试下安全强度
#https://www.ssllabs.com/ssltest/analyze.html?d=www.xxx.cn
```

```sh
#最后用的配置方式
server {
    listen 80;
    server_name service.xxx.cn;
    rewrite ^/(.*) https://service.xxx.cn/$1 permanent;
}
server {
    listen 443 ssl;
    server_name service.xxx.cn;
    location / {
        proxy_pass http://service.xxx.cn:8888;
    }
    charset utf-8;
    ssl_certificate /mine/data/https/xxx.cn.key.pem;
    ssl_certificate_key /mine/data/https/xxx.cn.key;
    ssl_dhparam /mine/data/https/dhparam.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128:AES256:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK;
}
```

