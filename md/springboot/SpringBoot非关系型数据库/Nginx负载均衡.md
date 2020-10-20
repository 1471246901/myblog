# Nginx 负载均衡

## 安装Nginx

ubuntu 安装

创建文件夹 `mkdir nginx`

### 下载nginx

```
wget https://nginx.org/download/nginx-1.14.0.tar.gz
```

解压到 nginx 文件夹

```java
tar -zxvf nginx-1.14.0.tar.gz
```

进入到 nginx 文件夹 并编译安装

```
cd nginx/nginx-1.14.0
./configure
make
make install
```

---

出现问题   `make: *** No rule to make target `build', needed by `default'. Stop.`

出现此种情况，是linux系统没有安装先决条件

**1、GCC——GNU编译器集合（GCC可以使用默认包管理器的仓库（repositories）来安装，包管理器的选择依赖于你使用的Linux发布版本，包管理器有不同的实现：yum是基于Red Hat的发布版本；apt用于Debian和Ubuntu；yast用于SuSE Linux等等。）**

**RedHat中安装GCC：**

yum install gcc

**Ubuntu中安装GCC：**

apt-get install gcc

**2、\**PCRE库（Nginx编译需要PCRE（Perl Compatible Regular Expression），因为Nginx的Rewrite模块和HTTP核心模块会使用到PCRE正则表达式语法。这里需要安装两个安装包pcre和pcre-devel。第一个安装包提供编译版本的库，而第二个提供开发阶段的头文件和编译项目的源代码，这正是我们需要的理由。）\****

**RedHat中安装\**\*\*PCRE\*\**\*：**

yum install pcre pcre-devel

**Ubuntu中安装\**\*\*PCRE\*\**\*：**

apt-get install libpcre3 libpcre3-dev

**3、\**zlib库（zlib库提供了开发人员的压缩算法，在Nginx的各种模块中需要使用gzip压缩。如同安装PCRE一样，同样需要安装库和它的源代码：zlib和zlib-devel。）\****

**RedHat中安装\**zlib\**：**

yum install zlib zlib-devel

**Ubuntu中安装\**\*\*zlib\*\**\*：**

apt-get install zlib1g zlib1g-dev

**4、\**OpenSSL\**\**库（在Nginx中，如果服务器提供安全网页时则会用到OpenSSL库，我们需要安装库文件和它的开发安装包（openssl和openssl-devel）。）\****

**RedHat中安装\**OpenSSL\**：**

yum install openssl openssl-devel

**Ubuntu中安装\**OpenSSL\**：（注：Ubuntu14.04的仓库中没有发现openssl-dev）：**

apt-get install openssl openssl-dev

---

另一种安装方法 

```
sudo apt install nginx
```

安装成功后启动nginx  `需要管理员权限`

```
whereis nginx
sudo /usr/sbin/nginx

```

### 配置nginx

进入nginx安装目录修改配置文件

```
gushiyu@gushiyunginx:/$ whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
gushiyu@gushiyunginx:/$ cd /etc/nginx
gushiyu@gushiyunginx:/etc/nginx$ ls
conf.d          koi-utf     modules-available  proxy_params     sites-enabled  win-utf
fastcgi.conf    koi-win     modules-enabled    scgi_params      snippets
fastcgi_params  mime.types  nginx.conf         sites-available  uwsgi_params
```

对nginx.conf 进行修改

```
vim nginx,conf
```

```
         upstream gushiyu.org{
                server 192.168.31.154:8080 weight=1;
                server 192.168.31.154:8081 weight=1;
        }
        server {
                listen 80;
                server_name localhost;
                location / {
                        proxy_pass http://gushiyu.org;
                        proxy_redirect default;
                }

        
```

重启nginx 和应用服务器

访问