

# Redis 安装

## Window 下安装

**下载地址：**https://github.com/MSOpenTech/redis/releases。

Redis 支持 32 位和 64 位。这个需要根据你系统平台的实际情况选择，这里我们下载 **Redis-x64-xxx.zip**压缩包到 C 盘，解压后，将文件夹重新命名为 **redis**。

![image-20200227183317937](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200227183317937.png)

打开文件夹，内容如下：

![fd](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200316190936101.png)

打开一个 **cmd** 窗口 使用 cd 命令切换目录到 **C:\redis** 运行：

```powershell
redis-server.exe redis.windows.conf
```

如果想方便的话，可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的。输入之后，会显示如下界面：

![image-20200227183358856](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200227183358856.png)

这时候另启一个 cmd 窗口，原来的不要关闭，不然就无法访问服务端了。

切换到 redis 目录下运行:

```powershell
redis-cli.exe -h 127.0.0.1 -p 6379
```

设置键值对:

```powershell
set myKey abc
```

取出键值对:

```powershell
get myKey
```

![AA](https://raw.githubusercontent.com/1471246901/myblog/master/img/image-20200227183450124.png)



----

## 在ubuntu上安装

### 第一种方法-apt安装

​	使用apt安装:

1. 升级软件管理模块apt：
   
   ``` shell 
   sudo apt-get update
   ```
   
2. 安装Redis服务:
   ```shell
   sudo apt-get install redis-server
   ```

3. 启动 Redis:

   ```shell
   redis-server [配置文件[redis.conf]]
   ```

4. 查看 redis 是否还在运行:

   ```shell
   ps -ef | grep redis    --> 查看进程
   netstat -an|grep 6379   --> redis的端口号是6379
   $redis-cli        --> 启动redis客户端
   ```

### 第二种方法-源文件编译方式

安装

- 下载压缩包：wget : ` http://download.redis.io/releases/redis-5.0.4.tar.gz`
- 解压：`tar xzf redis-5.0.4.tar.gz`
- 将解压后的文件移至  `/usr/local/redis/`（没有redis目录 就创建）：

``` shell 
 sudo mv ./redis-5.0.4 /usr/local/redis/
```
- 进入redis-5.0.4目录并生成编译

```shell
     cd redis-5.0.4
     sudo make
```

- 测试

  - 测试错误

    ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/1394443-20190804185845859-1930054888.png)

  - 解决方案

    ```shell 
    sudo apt install tcl 
    ```
    
  - 测试成功
  
    ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/1394443-20190804191337149-1261801629.png)

- 安装


```shell
sudo make install
```

  - 切换至安装目录 
    ``` shell 
    cd /usr/local/bin
    ```
    ![img](https://raw.githubusercontent.com/1471246901/myblog/master/img/1394443-20190804193124947-98967165.png)
    ``` tex
    redis-benchmark　　　　　redis性能测试工具
redis-check-rdb　　　　　 rdb文件检索工具　　　　　　
    redis-check-aof　　　　　 aof文件修复工具
    redis-cli　　　　　　　　　redis命令行客户端
    redis-server  　　　　　 redis服务器
    ```

   - 复制配置文件

　　　```shell
　　　sudo cp redis.conf /etc/redis/redis.conf
　　　```
　　　

   - 安装成功
