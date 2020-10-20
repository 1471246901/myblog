# [Linux安装RabbitMQ](https://my.oschina.net/liudandan/blog/3109810)

原创

[简小姐](https://my.oschina.net/liudandan)

[MQ](https://my.oschina.net/liudandan?tab=newest&catalogId=6661892)

2019/09/23 18:01

阅读数 1.8W

一、安装Erlang环境

1、在安装erlang之前先安装下依赖文件(这一步不要忘掉了， 不然后面./configure的时候要报错)：
 yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto
2.到erlang官网去下载erlang安装包
 wget -c http://erlang.org/download/otp_src_20.2.tar.gz
 
 接下来解压：
 tar -zxvf otp_src_20.2.tar.gz
 cd otp_src_20.2/
\3. 编译安装
 ./configure --prefix=/usr/local/erlang
 make && make install
4、测试安装是否成功：
  cd /usr/local/erlang/bin/ 
  ./erl
  halt(). 退出控制台
5、配置环境变量
 vim /etc/profile
 在末尾加入这么一行即可：export PATH=$PATH:/usr/local/erlang/bin　
 source /etc/profile
二、安装rabbitmq
　1、到官网下载最新安装包：http://www.rabbitmq.com/releases/rabbitmq-server/ 
 wget -c http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz
 解压：
 xz -d rabbitmq-server-generic-unix-3.6.15.tar.xz 
  tar -xvf rabbitmq-server-generic-unix-3.6.15.tar
2、配置rabbitmq的环境变量
  vim /etc/profile
   export PATH=$PATH:/usr/local/rabbitmq_server-3.6.15/sbin
  source /etc/profile
3、rabbitmq的基本操作：

　　　　启动：rabbitmq-server -detached

　　　　关闭：rabbitmqctl stop

　　　　查看状态：rabbitmqctl status

4、配置rabbitmq网页管理插件

　　　　启用插件： rabbitmq-plugins enable rabbitmq_management

 　　 访问管理页面：[http://192.168.?.?:15672](http://192.0.0.168/?.?:15672) 端口默认为15672 
 

或许你打不开  记得把防火墙关了

关闭防火墙命令：systemctl stop firewalld.service

又或者你这样:

\# 将mq的tcp监听端口和网页管理端口都设置成允许远程访问

firewall-cmd --permanent --add-port=15672/tcp

firewall-cmd --permanent --add-port=5672/tcp

systemctl restart firewalld.service

(我一般都是暴力操作直接关闭防火墙，因为我记得之前的胖领导给我说过linux 上的防防火墙没什么用)

 

s 

打开之后有个用户名和密码  

rabbitmq有一个默认的用户名和密码，guest和guest,但为了安全考虑，该用户名和密码只允许本地访问，如果是远程操作的话，需要创建新的用户名和密码；

\# root权限

rabbitmqctl add_user username passwd //添加用户，后面两个参数分别是用户名和密码 （liudan  liudan123）

rabbitmqctl set_permissions -p / username ".*" ".*" ".*" //添加权限  

rabbitmqctl set_user_tags username administrator //修改用户角色,将用户设为管理员

 