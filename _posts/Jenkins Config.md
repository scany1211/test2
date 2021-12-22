# Config

## 一. credential

### 应用场景

1.  用于全局global 设置， 所有application 均可用
2. specific Pipeline project/item某些项目或者item
3. 某个jenkins用户
4. 

###  类型

1. secret txt：a **token** such as an API token (e.g. a GitHub personal access token),
2. username/password:  某些组件的用户名密码username:password
3. secret file：存在某个文件中的secret
4. **SSH Username with private key**： 
5. Certifile：a [PKCS#12 certificate file](https://tools.ietf.org/html/rfc7292) and optional password,
6. **Docker Host Certificate Authentication** credentials.

### 作用模式

credential 被controller Jenkins instance (encrypted by the Jenkins instance ID)加密存贮在，只能被pipeline project通过他的instanceID 处理

无法从某个地方copy到另一个地方



### 配置crendiential

只有具有分配对应权限的用户才能配置。

#### ID

* 密码配置中ID，最好配置为有特殊含义的，比如`jenkins-user-for-xyz-artifact-repository`. inbuilt (default) credentials provider 可以使用这个ID（大小写，分隔符）；
* 如果不配置，则会分配一个全局的GUID，但是一旦分配就不可改变

#### scope

* system：作用到某个object

* global：Pipeline project/item "object" and all its descendent objects.

  ## 二. 配置时区

1. 场景：

Jenkins server所在时区与当前时区不一致，导致build中看到的时间对应不上。

2. 查看时区：

http://192.168.56.104:8081/systemInfo

或者curl http://fakeurl.com/systemInfo --user 'fakeuser':'fakepasswd'

3，修改user.timezone

your user configuration page, you can set the `User Defined Time Zone` to match your own.



## 三, CLI

通过脚本环境来管理jenkins



### 1. SSH 模式

默认情况下，ssh cli port是disabled，需要在global security 内启动该项。

![image-20211022155530713](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211022155530713.png)



再在当前user下将远端登陆机器的ssh pub填入，

ssh -l admin -p 45123  <Jenkins IP>  help

### 2. Jenkins client 模式



## 四、工作目录

### 1. master/standby node:

workspace: **JENKINS_HOME/workspace/\*folderName\*/\*subfolderName\*/\*projectName\***

### 2. agent node:

workspace:  **<rootDirectory>/workspace/<projectName>***



## 五. 多环境变量设置



### 1. tool 设置

页面上配置tool后，在stage 阶段引入tool 配置

``` groovy
pipeline {
        agent any

        stages {  
            stage ("first") {
                tools {
                   jdk "jdk-1.8.101"
                }
                steps {
                    sh 'java -version'
                }
            }
            stage("second"){
                tools {
                   jdk "jdk-1.8.152"
                }
                steps{
                    sh 'java -version'
                }
            }
       }
}
```



### 2. environment 

通过environment来设置：

``` groovy
        stage('Java MGMT - Build & Unit/Integration Test') {
            agent {
                label 'oldTni2'
            }
            environment {
                JAVA_HOME='/proj/teprod/tools/jdk/1.8/linux_x64'
                PATH='${env.JAVA_HOME}/bin:${env.PATH}'
            }
            steps {
                
            }
```

