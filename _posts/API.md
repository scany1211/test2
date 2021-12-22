# REST API

## 1. 作用

* 从Jenkins获取数据
* 触发build
* 拷贝或者创建jobs

如何使用

http://192.168.56.104:8081/api/ 可以查看更多



## 2. xml 

## 3. JSON 

## 4. Python

已经存在的python 包[JenkinsAPI](https://pypi.python.org/pypi/jenkinsapi), [Python-Jenkins](https://pypi.python.org/pypi/python-jenkins/), [api4jenkins](https://pypi.org/project/api4jenkins/), [aiojenkins](https://pypi.org/project/aiojenkins/) ，可以直接PIP安装获取函数。提供如下功能

- Query the test-results of a completed build， 获取已完成build数据
- Get objects representing the latest builds of a job， job中的对象
- Search for artifacts by simple criteria， 搜索artifacts
- Block until jobs are complete， Block以完成job
- Install artifacts to custom-specified directory structures， 安装artifacts
- Authentication support for Jenkins instances， 认证
- Ability to search for builds by subversion revision。 subversion
- Ability to add/remove/query Jenkins agents， 管理agents
- 

## 