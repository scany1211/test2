# 场景

job1中生成一个参数（只能是Jenkins enviroment variables），需要传递给downstream 的job2, 并在job2中展示.



# 方法1

1.  安装“Parameterized Trigger Plugin”.
2. 在upstream job1中配置如下：

**Job 1(upstream job) > Post-Build Actions > Select ”Trigger parameterized build on other projects”**

[![Screen Shot 2015-02-09 at 4.04.49 pm](https://itisatechiesworld.files.wordpress.com/2015/02/screen-shot-2015-02-09-at-4-04-49-pm.png?w=538&h=246)](https://itisatechiesworld.files.wordpress.com/2015/02/screen-shot-2015-02-09-at-4-04-49-pm.png)

选择job1中的Build_Number传递给下一个job. 并对其分配一个变量 ‘SOURCE_BUILD_NUMBER’.



3. 在job2中定义变量SOURCE_BUILD_NUMBER

![enter image description here](https://i.stack.imgur.com/lW9EG.jpg)



# 方法2





在方法1中，将job1中的变量写入文件，在job2中配置从文件中读取参数

'Add post-build action' -> 'Trigger parameterized build...' then  selecting 'Add Parameters' -> 'Parameters from properties file'.

![image-20211119111744326](C:\Users\songhyan\AppData\Roaming\Typora\typora-user-images\image-20211119111744326.png)