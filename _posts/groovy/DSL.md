# job DSL

plugin: Job dsl plugin

作用：将job转换为code

 Writing such a file is feasible without being a Jenkins  expert as the configuration from the web UI translates intuitively into  code.

[![configuration form](https://cdn.jsdelivr.net/gh/jenkinsci/job-dsl-plugin@master/docs/images/pipeline.png)](https://github.com/jenkinsci/job-dsl-plugin/blob/master/docs/images/pipeline.png)

The job configuration above can be generated from the following code.

```
pipelineJob('job-dsl-plugin') {
  definition {
    cpsScm {
      scm {
        git {
          remote {
            url('https://github.com/jenkinsci/job-dsl-plugin.git')
          }
          branch('*/master')
        }
      }
      lightweight()
    }
  }
}
```



# pipeline DSL

plugin：Pipeline plugin

作用：通过DSL实现pipeline，可以与Job DSL结合

Pipeline is often the better choice for creating complex automated  processes. Job DSL can be used to create Pipeline and Multibranch  Pipeline jobs.

Job DSL的DSL用于创建形成管道的作业，Pipeline Plugin的DSL定义了管道本身。

# Configuration as code

The Configuration as Code plugin can be used to manage the global system configuration of Jenkins. It comes with an integration for Job DSL to  create an initial set of jobs.

# 配置解析

https://jenkinsci.github.io/job-dsl-plugin/#