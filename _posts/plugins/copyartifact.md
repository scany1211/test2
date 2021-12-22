



1. 在A节点新增"构建后操作"，选择"归档成品(Archive the artifacts)"，在"Files to archives"中填写归档文件的名称，这里以当前任务的Workspace目录开始，比如我们的文件完成路径是

   ```
   /var/lib/jenkins/workspace/job_a/dist.zip
   ```

   ，那么归档文件的名称填写"dist.zip"即可，如果有多个文件，使用英文逗号","隔开，如图：

   ![image-20211117134743011](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211117134743011.png)

2. 在B节点任务中增加构建步骤，选择"Copy artifacts from another project"，然后填写需要复制的文件，以及保存的路径，如图

   ![image-20211117134835192](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211117134835192.png)

3. `Project name`: 指定父级任务的名称

4. `Which build`：指定需要基于父级任务的哪一次构建，这里选择了最新一次成功的构建

5. `Artifacts to copy`：指定了需要拷贝的文件名称，多个以英文逗号","隔开，留空的话会复制上级任务的所有归档文件

6. `Artifacts not to copy`：指定不需要拷贝的文件名称

7. `Target directory`：指定归档文件存放的目录

8. `Parameter filters`：用于参数过滤，暂时忽略

运行任务，我们会在日志中看到类似下面的输出，成功从上级任务中复制了两个归档文件。

![image-20211117141117996](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211117141117996.png)

进入服务器目录中，我们也可以看到这两个归档文件

![img](https://ask.qcloudimg.com/http-save/yehe-1046831/v2f5mgpyda.png?imageView2/2/w/1620)



# 例子

## Use with declarative pipelines

One example:

```
stages {
    stage('Copy Archive') {
         steps {
             script {
                step ([$class: 'CopyArtifact',
                    projectName: 'Create_archive',
                    filter: "packages/infra*.zip",
                    target: 'Infra']);
            }
        }
    }
...
}
```

What that is doing:

- Go to the project/workspace named "Create_archive".
- Look in the folder "packages" for the file(s) "infra*.zip".
- Copy that file(s) into the folder "Infra", in the local workspace. Folder will be created if it doesn’t already exist.



