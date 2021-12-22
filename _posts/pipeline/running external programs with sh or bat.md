# running external programs with sh or bat





**sh **: 在linux中调用sh

**bat**：调用windows 的bat



```
pipeline {
    agent { label 'master' }
    stages {
        stage('build') {
            steps {
                echo "Hello World!" // jenkins自己的echo
                sh "echo Hello from the shell" //sh 中的echo
                sh "hostname"
                sh "uptime"
            }
        }
    }
}
```

The result looks like this:



```
Started by user Gabor Szabo
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/simple-pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] echo
Hello World!
[Pipeline] sh
[simple-pipeline] Running shell script
+ echo Hello from the shell
Hello from the shell
[Pipeline] sh
[simple-pipeline] Running shell script
+ hostname
s17
[Pipeline] sh
[simple-pipeline] Running shell script
+ uptime
 17:15:35 up 3 days,  1:59,  0 users,  load average: 0.00, 0.00, 0.00
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```



## MS Windows

A few examples in Windows:

Show the name of the computer (a bit like hostname on Unix):



```
bat 'wmic computersystem get name'
```

Print out the content of the PATH environment variable as seen by Windows.



```
bat 'echo %PATH%'
```

Of course we could have printed the PATH environment variable without invoking an external call:



```
echo env.PATH
```



## Print out all the environment variables seen by Windows.



```
echo bat(returnStdout: true, script: 'set')
```



## Get the disk size of a local disk



```
script {
    def disk_size = sh(script: "df / --output=avail | tail -1", returnStdout: true).trim() as Integer
    println("disk_size = ${disk_size}")
}
```



Full examples:

**examples/jenkins/list_disk_size.Jenkinsfile**

```
pipeline {    agent { label 'master' }    stages {        stage('build') {            steps {                script {                    def disk_size = sh(script: "df / --output=avail | tail -1", returnStdout: true).trim() as Integer                    println("disk_size = ${disk_size}")                }            }        }    }}   
```



## Get the disk size of a remote disk



```
script {
    def disk_size = sh(script: "ssh remote-server df / --output=avail | tail -1", returnStdout: true).trim() as Integer
    println("disk_size = ${disk_size}")
}
```