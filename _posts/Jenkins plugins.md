# plugins

## 工作原理



## 安装方式

### 1. 页面自动安装

### 2. 手动安装

将hpi下载放到Jenkins的home 路径(/var/lib/jenkins---通过api systemInfo查看)

Copy the downloaded `.hpi`` file into the `JENKINS_HOME/plugins` directory on the Jenkins controller (for example, on Debian systems `JENKINS_HOME` is generally `/var/lib/jenkins`).

### 3. 命令安装



```
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin SOURCE ... [-deploy] [-name VAL] [-restart]

Installs a plugin either from a file, an URL, or from update center.

 SOURCE    : If this points to a local file, that file will be installed. If
             this is an URL, Jenkins downloads the URL and installs that as a
             plugin.Otherwise the name is assumed to be the short name of the
             plugin in the existing update center (like "findbugs"),and the
             plugin will be installed from the update center.
 -deploy   : Deploy plugins right away without postponing them until the reboot.
 -name VAL : If specified, the plugin will be installed as this short name
             (whereas normally the name is inferred from the source name
             automatically).
 -restart  : Restart Jenkins upon successful installation.
```



## 删除plugins

* 在页面操作后，需要重启才能生效
* 如果plugins有依赖，重启会报错
* 页面删除，无法删除对应的配置文件
  * 页面无法删除的数据，可以通过这个删除 -----Navigate to **Manage Jenkins** and then click on **Manage Old Data** to review and remove old data

disable plugins： Jenkins认识这个plugins，但是不会start它。no extensions contributed from this plugin will be visible.

## 常用plugins

### copy artifcats

通过名字索引另外一个project或者job， 拷贝artifacts

```
copyArtifacts projectName: 'myproject'
```

* 需要保证project名字是唯一的，且没有subproject。

语法：

- To copy artifacts from the latest stable build of "sourceproject"

  ```
  copyArtifacts(projectName: 'sourceproject');
  ```

- To copy artifacts from the specific build of "downstream"

  ```
  def built = build('downstream');  // https://plugins.jenkins.io/pipeline-build-step
  copyArtifacts(projectName: 'downstream', selector: specific("${built.number}"));
  ```

- Parameters of copyArtifacts

  | parameter            | type          | description                                                  |
  | -------------------- | ------------- | ------------------------------------------------------------ |
  | projectName          | string        | the name of project (required)                               |
  | selector             | BuildSelector | the selector to select the build to copy from. If not specified, latest stable build is used. |
  | parameters           | string        | comma-separated name and value pairs (name1=value1,name2=value2) to filter the build to copy from. |
  | filter               | string        | ant-expression to filter artifacts to copy                   |
  | excludes             | string        | ant-expression to exclude artifacts to copy                  |
  | target               | string        | target directory to copy to                                  |
  | flatten              | boolean       | ignore directory structures of artifacts                     |
  | optional             | boolean       | do not fail the step even if no appropriate build is found.  |
  | fingerprintArtifacts | boolean       | fingerprint artifacts to track builds using those  artifacts. false for default if the parameter isn’t specified (Snippet  Generator defaults this to true and specifies the parameter). |
  | resultVariableSuffix | boolean       | useless for pipelines                                        |

- selectors

  | name             | feature                                    |
  | ---------------- | ------------------------------------------ |
  | lastSuccessful   | Latest successful build                    |
  | specific         | Specific build                             |
  | permalink        | Specified by permalink                     |
  | lastCompleted    | completed build (ignoring build status)    |
  | latestSavedBuild | Latest saved build (marked "keep forever") |
  | buildParameter   | Specified by a build parameter             |
  | upstream         | Upstream build that triggered this job     |

使用建议：

* 压缩之后在copy
* 某些版本有权限问题，需要设定权限

### Folder plugins

允许用户在 users to create "folders" to organize jobs，Users can define custom taxonomies (e.g. by project type, organization  type). Folders are nestable and you can define views within folders.

如果多个相同名字的project，但是在不同的目录下。

* 绝对路径：

  * to reference a project in the root of your Jenkins instance:

    ```
    /myproject
    ```

    Or, to reference a project in a subfolder:

    ```
    /myfolder/myproject
    ```

* 相对路径：

  * ```
    /thatproject
    /folder/someproject
    /folder/subfolder/myproject
    /folder/subfolder/anotherproject
    ```

    In a Pipeline Script for `/folder/subfolder/myproject`, you could refer to `/folder/subfolder/anotherproject` using this relative path:

    ```
    anotherproject
    ```

    And you could refer to `/folder/someproject` using this relative path, where `..` means to look in the parent folder:

    ```
    ../someproject
    ```

    And you could refer to `/thatproject` using this relative path:

    ```
    ../../thatproject
    ```

* project 内的子目录： 某些有多个分支的project，或者maven project，会有多个目录

  * ### Maven projects

    You can refer to an entire Maven project:

    ```
    mymavenproject
    ```

    Or to a group within a Maven project:

    ```
    mymavenproject/my.group
    ```

    Or to a particular module:

    ```
    mymavenproject/my.group$MyModule
    ```

    ### Matrix projects

    You can refer to all configurations of a Matrix project:

    ```
    mymatrixproject
    ```

    Or to a particular configuration, restricted by a axis:

    ```
    mymatrixproject/someaxis=somevalue
    ```

    Or restricted by multiple axes:

    ```
    mymatrixproject/someaxis=somevalue,anotheraxis=anothervalue
    ```

    ### Multibranch Pipelines

    You can refer to a particular branch:

    ```
    mymultibranchproject/mybranch
    ```

    ## Name encoding

    Special characters in paths should be URL-encoded. For example, if your Multibranch Pipeline has a branch with a slash in it (`feature/myfeature`), replace the slash with `%2F`:

    ```
    mymultibranchproject/feature%2Fmyfeature
    ```

    

