### stash: 保存临时文件

stash 步骤可以将一些文件保存起来，以便被同一次构建的其他步骤或阶段使用。如果整个 pipeline 的所有阶段在同一台机器上执行，则 stash 步骤是多余的。所以，通常需要 stash 的文件都是要跨 Jenkins node 使用的。stash 步骤会将文件存储在 tar 文件中，对于大文件的 stash 操作将会消耗 Jenkins master 的计算资源。Jenkins 官方文档推荐，当文件大小为  5~100MB时，应该考虑使用其他替代方案。stash 步骤的参数列表如下：

- name: 字符串类型，保存文件的集合的唯一标识；
- allowEmpty: 布尔类型，允许 stash 内容为空；
- excludes: 字符串类型，将哪些文件排除。如果排除多个文件，则使用逗号分隔。留空代表不排除任何文件；
- includes: 字符串类型，stash 哪些文件，留空代表当前文件夹下的所有文件；
- useDefaultExcludes: 布尔类型，如果为 true, 则代表使用 Ant 风格路径默认排除文件列表。

除了 name 参数，其他的参数都是可选的。excludes 和 includes 使用的是 Ant 风格路径表达式。

### unstash: 取出之前 stash 的文件

unstash 步骤只有一个 name 参数，即 stash 时的唯一标识。通常 stash 与 unstash 步骤同时使用。以下是完整示例：

```
pipeline {
    agent any
    stages {
        stage('stash') {
            agent {label: "master"}
            steps {
                writeFile file: "a.txt", text: "$BUILD_NUMBER"
                stash(name: "abc", includes: "a.txt")
            }
        }
        stage('unstash') {
            agent {label: node2}
            steps {
                script {
                    unstash("abc")
                    def content = readFile("a.txt")
                    echo "${content}"
                }
            }
        }
    }
}
```

stash 步骤在 master 节点上执行，而 unstash 步骤在 node2 节点上执行。