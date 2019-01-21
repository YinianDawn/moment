# centos7-docker运行gitlab

## 1. 准备配置文件gitlab.rb放置于/gitlab/config文件夹下
```
external_url 'https://gitlab.xxx.cn'
nginx['enable'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.xxx.cn.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.xxx.cn.key"
```
## 2. 执行docker命令配置启动容器
```bash
#!/bin/bash
docker stop gitlab
docker rm gitlab
docker run --detach \
    --hostname gitlab.xxx.cn \
    --publish 5555:80 -p 5556:443 -p 5557:22 \
    --name gitlab \
    --volume /gitlab/config:/etc/gitlab \
    --volume /gitlab/logs:/var/log/gitlab \
    --volume /gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
## 3.如果已经开启5556端口，即可访问，或者配置nginx的https代理也可
```
server {
    listen 443 ssl;
    server_name gitlab.xxx.cn;
    location / {
        # /etc/hosts文件已将该域名指向本地
        proxy_pass https://gitlab.xxx.cn:5556;
    }
    charset utf-8;
    ssl_certificate /https/xxx.cn.key.pem;
    ssl_certificate_key /https/xxx.cn.key;
    ssl_dhparam /https/dhparam.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets on;
    ssl_protocols SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128:AES256:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK;
}

```

#### 注意：docker restart gitlab会使得gitlab启动失败，不得已的办法是，每次重启都重新执行最上面的脚本，只要第一次生成的data目录不动，还是可以继续使用的。囧
