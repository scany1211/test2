# Fingerprint

## 作用

## 场景：

TOP project ---> Middle project ------> Bottom project

TOP依赖MIDDLE, MIDDLE依赖BOTTOM。当BOTTOM某个包做了修改，TOP, MIDDLE需要知道他们**能调用哪个包， 并且哪个TOP,MIDDLE在调用这个包**

**能调用哪个版本的包 **

**被哪个项目哪个版本调用** 

## 原理

Jenkins只存储该文件的MD5 checksum, 存在一个数据库中，每条数据，会记录哪个project的哪个build在使用， 该数据库会在**每次build或者每个fingerprint配置时更新**， 存储路径

**$JENKINS_HOME/fingerprints**







