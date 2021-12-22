# 声明式

# 脚本式： pipeline大小有极限

## 1. when

* 作用：判断是否执行某个stage
* 作用：单个条件时，为true才执行；多个条件时，全为true才执行。子嵌套条件包含 ```not, allOf, or anyOf ``` 

```groovy
stages {
    stage('Deploy') {
        when { 
            expression {
                return Deploy
            }
        }
        steps {
            echo "Hello, bitwiseman!" // Step executes only when Deploy is true

        }
    }
}
```

* 常用的判断：

  * branch：当分支名字满足某个pattern时，  example: `when { branch pattern: "release-\\d+", comparator: "REGEXP"}`、 when { branch 'master' }

  

  * **定义的变量都会被转换成string**， 包括false, true等。需要通过env.xxx.toBoolean() 来转换改变量为boolen

      

    ```groovy
          enviroment { Isinstall = 'false' }
          ....
              when { 
                    expression {
                        env.Isinstall.toBoolean()
                    }
                }
    ```

  

## 2. Enviroment

#### 使用场景

在全局或者某些stage设定key-value 键对；还可以用过credentials() 获取预先定义的credentials；



## 3. Parameters

#### 使用场景：

​	需要用户在启动pipeline的时候提供的一个参数

#### 格式：

- string

  A parameter of a string type, for example: `parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }`

- text

  A text parameter, which can contain multiple lines, for example: `parameters { text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '') }`

- booleanParam

  A boolean parameter, for example: `parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') }`

- choice

  A choice parameter, for example: `parameters { choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '') }`

- password

  A password parameter, for example: `parameters { password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password') }`

#### 使用方式：

初次build会使用默认值，后续可以输入或者修改参数进行build。

![image-20211026153150648](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211026153150648.png)



## 4. Agent

#### 使用场景

指定整个pipeline或某个stage执行的节点，top-level agent必须定义。

#### 格式

``` groovy
agent {
	type_name: {
	
	}
}
```



#### 使用方式：

* any

任何类型的agent . For example: `agent any`

* none

When applied at the top-level of the `pipeline` block no global agent will be allocated for the entire Pipeline run and each `stage` section will need to contain its own `agent` section. For example: `agent none`

* label

带有指定label的agent，可以是node，docker等。. For example: `agent { label 'my-defined-label' }`

可以与或者或。    For example: `agent { label 'my-label1 && my-label2' }` or `agent { label 'my-label1 || my-label2' }`

* node

  与label很像，但是还可以定义更多参数，如customWorkspace.

`agent { node { label 'labelName' } }` behaves the same as `agent { label 'labelName' }`, but `node` allows for additional options (such as `customWorkspace`).

* docker

**需要提前安装docker plugins , docker pipeline**

==docker运行的节点，可以在配置node时，指定是否接受docker-based pipeline, 也可以通过match label指定运行的节点。==

docker中的参数包含：

 	`label` parameter.  
 	
 	`alwaysPull` ：which will force a `docker pull` even if the image name is already present. 

​		`args`: -v , -e ...

For example: `agent { docker 'maven:3.8.1-adoptopenjdk-11' }` or

```
agent {
    docker {
        image 'maven:3.8.1-adoptopenjdk-11'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}
```

​		`registryUrl` : 指定registry

​		`registryCredentialsId`：指定registry的 credentials. 

For example:

```
agent {
    docker {
        image 'myregistry.com/node'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
```

* dockerfile

通过代码源中的Dockfile去build 一个container。为了能够使用这种方式，, the `Jenkinsfile` must be loaded from either a **Multibranch Pipeline** or a **Pipeline from SCM**. Conventionally this is the `Dockerfile` in the root of the source repository: `agent { dockerfile true }`. If building a `Dockerfile` in another directory, use the `dir` option: `agent { dockerfile { dir 'someSubDir' } }`. If your `Dockerfile` has another name, you can specify the file name with the `filename` option. You can pass additional arguments to the `docker build …` command with the `additionalBuildArgs` option, like `agent { dockerfile { additionalBuildArgs '--build-arg foo=bar' } }`. For example, a repository with the file `build/Dockerfile.build`, expecting a build argument `version`:

```
agent {
    // Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        additionalBuildArgs  '--build-arg version=1.0.2'
        args '-v /tmp:/tmp'
    }
}
```

# 两者区别

* 声明式
  * **pipeline { } ** 将要做的动作圈起来
  * 有**stages 、steps **更细致的阶段划分
  * 复杂逻辑编写繁琐，如循环则需要使用**script** { for()}
* 脚本式
  * **node ** { }将要做的动作圈起来
  * 没有stages. steps等阶段
  * 只支持**stage** {}
  * 编写复杂逻辑

