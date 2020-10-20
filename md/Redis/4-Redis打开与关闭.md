# Redis打开与关闭

## 打开Redis

- 服务端 redis-server 

- 客户端 redis-cli 

  ```
  -a [密码]
  ```

## 关闭

- 第一种关闭方式`强制关闭`

  通过查询PID`进程ID`

  通过命令杀死进程   开发中不可能使用

  ```shell
  ps -ef | grep -i redis
  kill -9 PID
  ```

- 第二种关闭方式`正常关闭`

  通过客户端执行`shutdown` 命令关闭

  如果设置了密码,还需通过密码登陆后再能关闭

  ```shell
  redis-cli shutdown 
  ```

  

