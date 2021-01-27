# zookeeper单机安装

## 下载安装包

在官网下载zookeeper和jdk的安装包

 [zookeeper下载地址](https://zookeeper.apache.org/releases.html) 

[jdk8 下载地址](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

下载完成得到 `jdk:jdk-8u131-linux-x64.tar.gz`,` zookeeper:zookeeper-3.4.10.tar.gz`

将其上传到服务器，或使用服务器下载 `rz`

## 安装JDK8

### **解压到指定目录** 

自己指定目录解压

```powershell
tar -xzvf jdk-8u131-linux-x64.tar.gz
```

进入解压后的目录，使用`pwd`得到绝对路径，复制下来

### **配置环境变量**

依照 [Linux配置环境变量](..\java\Linux配置环境变量方式.md) 进行配置

```shell
// vim 打开 ~/.profile文件 位于个人文件夹下
vi .profile
// 文件中加入如下内容 这里就是pwd获取的内容
JAVA_HOME=/home/zookeeper/jdk1.8.0_131  
export JAVA_HOME
PATH=$JAVA_HOME/bin:$PATH
export PATH
// 使环境变量生效
source .profile
```

使用 java -version 和 javac 来验证是否配置成功

## 安装zookeeper

### 解压到指定目录

```shell
// 解压zookeeper
tar -xzvf zookeeper-3.4.10.tar.gz
```

### 准备配置文件

进入到解压的文件夹下的 conf文件夹 `zoo_sample.cfg` 文件时zookeeper配置文件的样例 ，将其复制为 `zoo.cfg` 文件

```java
// 进入conf目录
cd /home/zookeeper/zookeeper-3.4.10/conf
// 复制配置文件
cp zoo_sample.cfg zoo.cfg
```

### 准备持久化文件夹

在解压的文件夹下创建 data文件夹 用于存储zookeeper中数据的内存快照、及事物日志文件

```
# 在zookeeper-3.4.10文件夹下创建data
mkdir data
# 获取data文件夹的路径，复制下来
pwd
```

### 修改配置文件

进入到conf文件夹下，修改刚刚复制的 zoo.cfg

```
cd conf
vim zoo.cfg
```

修改内容为

```
// /home/zookeeper/zookeeper-3.4.10/data 就是刚刚创建的文件夹的路径
dataDir=/home/zookeeper/zookeeper-3.4.10/data
```

## 启动zookeeper

进入到安装目录下的bin目录下，zookeeper的运行脚本是 `zkServer.sh ` 

### 启动zookeeper

```
//启动：zkServer.sh start
```

### 停止以及查看状态

```shell
//停止：zkServer.sh stop
//查看状态：zkServer.sh status
```

