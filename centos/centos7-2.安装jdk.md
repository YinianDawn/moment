# centos7-安装jdk

```sh
#下载jdk，下载地址，可以去jdk官网获得，，我这个肯定过期了
wget http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.tar.gz?AuthParam=1522861761_21c14f121595d784b3c5e5cd8a83622b
#重命名
mv jdk-8u162-linux-x64.tar.gz?AuthParam=1522861761_21c14f121595d784b3c5e5cd8a83622b jdk-8u162-linux-x64.tar.gz
#解压到/usr/local
tar -zxvf jdk-8u162-linux-x64.tar.gz -C /usr/local/
rm -rf jdk-8u162-linux-x64.tar.gz
#创建软连接，就可以用/usr/local/jdk来使用了，将来换版本直接把软连接换了就好了
ln -s /usr/local/jdk1.8.0_162 /usr/local/jdk
#配置环境变量
echo '' >> /etc/profile
echo 'export JAVA_HOME=/usr/local/jdk' >> /etc/profile 
echo 'export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar' >> /etc/profile 
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile
#刷新配置文件
source /etc/profile
#可以直接用java命令了
java -version
```