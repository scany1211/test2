# Jenkins安装

## 依赖

java

## 配置文件

 `/etc/default/jenkins` 

可以修改默认8080端口

查看一些系统配置：

http://192.168.56.104:8081/systemInfo

* 安装完成后的密码配置：/var/lib/jenkins/secrets/initialAdminPassword

## slave节点安装

### 必须安装的软件

* java8/java11
* 

### 1. 添加node

#### 启动方式

Launch agent agents via SSH
 通过安全SSH连接发送命令来启动从节点。需要从主服务器访问从服务器，并且您必须提供可以在目标计算机上登录的帐户。不需要root权限。

- 主机：即从节点的主机ip

- Credentials：凭据，如果没有添加过任务凭据，可以点击添加，凭据的用户名和密码分别是从节点访问的用户名和密码。

- Host Key Verification Strategy
   主机密钥验证策略，控制Jenkins如何在连接时验证远程主机提供的SSH密钥（注意，这里远程主机，值的是从节点，因为现在配置的是从节点的属性）

  1. known hosts file verification strategy
      已知主机文件验证策略

     ​	保证jenkins 用户的(~/.ssh/known_hosts)有对应主机的信息，或者通过-Dhudson.plugins.sshslaves.verifiers.KnownHostsFileKeyVerificationStrategy.known_hosts_file=PATH_TO_FILE 修改known_hosts的文件配置。

  2. manually provided key verification strategy
      手动提供密钥验证策略，需要手动提供<key_name>.pub

     ![image-20211025163854005](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211025163854005.png)

  3. manually trusted key verification strategy
      手动验证密钥验证策略
      选择 **Manually Trusted Key Verification Strategy**，这会在完成配置后，第一次连接从节点时要求我们手动确认当前连接的从节点的身份。说明一下，Host Key 验证是用来防止中间人攻击的

#### Node property

Deferred wipeout : 意思是**deletion takes place asynchronously** ， 即先rename workspace directory to a temporary directory name.再启动一个后台任务来删除这个临时任务。

**对于cloud环境，由于任务结束后node会消失，因此后台任务也会中断，默认需要disable这个功能**

### 2. 添加docker node

**docker image必须有java和sshd**

* 原理： Jenkins master通过REST API 连接docker。

* docker 主机启动docker时，需要暴露一个api port：

  ```
  ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
  sudo systemctl daemon-reload
  sudo service docker restart
  curl http://localhost:4243/version
  
  ```

