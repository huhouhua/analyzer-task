# insight-chart是什么？
是一个定义定时扫描任务的项目，并且这些项目集成了sonarQube插件， 提供给[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)组件来读取定义的任务并完成代码扫描工作，自动扫描文件变更同步机制，这个仓库可以定义在任何的组里面，也可以将扫描任务文件独立出来，定义在不同的项目里面。

# 准备工作

**taskFilePath**可以定义在任何的项目里面，但是需要在[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)组件运行时，在**application-prod.yaml**文件里指定以下参数。
 ``` yaml
 git-repository:
  repository:
    url: "git@172.17.189.70:sonar/insight-chart.git" #git仓库地址，也就是提供扫描任务的仓库，这里指定为insight-chart仓库。
    branch: "develop" #扫描的仓库分支
    taskFilePath: "/examples/default.yaml" #具体扫描的任务文件
 ```

# code-analyzer如何部署
### 1. 准备工作
  请点击[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)，查看安装docker、sonarQube、相关参数解释与配置等。
### 2. 镜像拉取
``` shell
 docker pull 172.17.162.231/devops/analyzer-scheduler:1.0
 docker pull 172.17.162.231/devops/analyzer-job:1.0
```
### 3. 运行
``` shell
  docker run -d --name analyzer-scheduler -v /root/analyzer/application-prod.yaml:/app/application-prod.yaml -v /var/run/docker.sock:/var/run/docker.sock 172.17.162.231/devops/analyzer-scheduler:1.0
```

# 样例说明
### 
