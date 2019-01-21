# centos7-nginx配置文件

```sh
#配置文件放这里
mkdir -p /mine/data/nginx/conf/etc
ln -s /mine/data/nginx/conf/etc /usr/local/nginx/conf/etc
#修改原本配置文件，使其包含我们的配置文件
sed -i "s|#gzip  on;|#gzip  on;\n\n    include etc/start.config;\n    include etc/*.conf;\n    include etc/end.config;|g" /usr/local/nginx/conf/nginx.conf
#起始文件
echo "
server {
    listen 80;
    server_name _;
    return 404;
}" > /usr/local/nginx/conf/etc/start.config
#末尾文件
echo "
" > /usr/local/nginx/conf/etc/end.config

systemctl restart nginx
```

