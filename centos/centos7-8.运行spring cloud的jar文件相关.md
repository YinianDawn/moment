# centos7-运行spring cloud的jar文件相关

```sh
#!/bin/sh
name=xxx
path=/mine/app
case "$1" in
    start)
        #启动
        nohup /usr/local/jdk/bin/java -jar $path/$name.jar > /dev/null 2>&1 &
        ;;
    stop)
        #停止
        ps -ef | grep "$name.jar" | grep -v grep | awk '{print $2}' | xargs kill -9
        ;;
    check)
        #检查并执行
        count=`ps -ef | grep "$name.jar" | grep -v grep | awk '{print $2}' | wc -l`
        if [ $count -lt 1 ]; then
            nohup /usr/local/jdk/bin/java -jar $path/$name.jar > /dev/null 2>&1 &
        fi
        ;;
    *)
        echo 'Please use start or stop or check as first argument'
        ;;
esac
```

