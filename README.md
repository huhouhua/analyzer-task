# insight-chart是什么？
insight-chart是一个用于定义定时扫描任务的仓库，并且这些任务（仓库）集成了sonarQube， 提供给[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)组件来读取定义的任务并完成代码扫描工作，自带自动扫描文件变更同步机制，这个仓库可以定义在任何的组里面，也可以将扫描任务文件独立出来，定义在不同的项目里面，code-analyzer是如何使用的请点击[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)。

# 准备工作

在[`调度配置`](http://172.17.189.70/sonar/code-analyzer#1%E8%B0%83%E5%BA%A6%E9%85%8D%E7%BD%AE)组件运行时，在**application-prod.yaml**文件里指定以下参数，**taskFilePath**可以定义在任何的项目里面。
 ``` yaml
 git-repository:
  repository:
    url: "git@172.17.189.70:sonar/insight-chart.git" #git仓库地址，也就是提供扫描任务的仓库，这里指定为insight-chart仓库。
    branch: "develop" #扫描的仓库分支
    taskFilePath: "/tasks/develop.yaml" #具体扫描的任务文件
 ```

# code-analyzer如何部署
### 1. 准备工作
  请点击[`环境准备`](http://172.17.189.70/sonar/code-analyzer#%E4%B8%80%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)，查看安装docker、sonarQube、相关参数解释与配置等。
### 2. 镜像拉取
``` shell
 docker pull 172.17.162.231/devops/analyzer-scheduler:1.0
 docker pull 172.17.162.231/devops/analyzer-job:1.0
```
### 3. 运行
``` shell
  docker run -d --name analyzer-scheduler --restart always  -v /root/analyzer/application-prod.yaml:/app/application-prod.yaml -v /var/run/docker.sock:/var/run/docker.sock 172.17.162.231/devops/analyzer-scheduler:1.0
```
# 扫描文件样例说明
### 样例一：指定具体的参数，不采用全局参数。
样例文件：[`default`](http://172.17.189.70/sonar/insight-chart/-/blob/develop/examples/default.yaml)
``` yaml
global:
  sonar:
    dockerFilePath: release/docker/SonarDockerfile
    nativeFilePath: sonar-project.properties
    mode: dockerfile
  parallel: 5 #并行数，每一组group的项目一次扫描多少个项目，如果group没有指定parallel参数的话，那么会默认使用这个并行数。
  triggerTimeCron: "0 0 6 ? * MON" #任务扫描时间,每周一上午6点开始扫描
  repo:
    branch: develop #扫描的仓库分支，如果projects没有指定branch参数的话，那么会默认使用这个分支。
groups: #组定义，可以定义多个组，放在最前面的组，最先开始扫描。
- name: app-xxx #组名，可以重复，可以简单的定义为比如xxx业务下所有的前端服务，用英文表示
  parallel: 4 #并行数，一次扫描，扫描多少个项目。
  description: "app-xxx example groups,this is for testing!" #组描述
  projects: # 项目定义
  - name: app-web #项目名，可以重复
    mode: dockerfile #项目扫描方式，默认使用dockerfile
    description: "app-web is front-end project！"  #项目描述
    repo: #仓库定义
      url: git@172.17.189.70:xxxxx-groups/app-xxx.git #仓库地址，只支持ssh方式。
      branch: develop #仓库扫描的分支
      sonarFilePath: release/docker/SonarDockerfile #dockerfile 扫描的文件路径，这个路径是任务仓库里面的路径。
```
### 样例二：扫描每个项目不同分支
样例文件：[`multipleBranch`](http://172.17.189.70/sonar/insight-chart/-/blob/develop/examples/multipleBranch.yaml)
``` yaml
global:
  sonar:
    dockerFilePath: release/docker/SonarDockerfile
    nativeFilePath: sonar-project.properties
    mode: dockerfile
  parallel: 5 #并行数，每一组group的项目一次扫描多少个项目，如果group没有指定parallel参数的话，那么会默认使用这个并行数。
  triggerTimeCron: "0 0 6 ? * MON" #任务扫描时间,每周一上午6点开始扫描
  repo:
    branch: develop #扫描的仓库分支，如果projects没有指定branch参数的话，那么会默认使用这个分支。
groups: #组定义，可以定义多个组，放在最前面的组，最先开始扫描。
- name: app-xxx #组名，可以重复，可以简单的定义为比如xxx业务下所有的前端服务，用英文表示
  parallel: 4 #并行数，一次扫描，扫描多少个项目。
  description: "app-xxx example groups,this is for testing!" #组描述
  projects: # 项目定义
  - name: app-web #项目名，可以重复
    mode: dockerfile #项目扫描方式，默认使用dockerfile
    description: "app-web is front-end project！"  #项目描述
    repo: #仓库定义
      url: git@172.17.189.70:xxxxx-groups/app-xxx.git #仓库地址，只支持ssh方式。
      branch: develop #仓库扫描的分支
      sonarFilePath: release/docker/SonarDockerfile #dockerfile 扫描的文件路径，这个路径是任务仓库里面的路径。
  - name: app-service
    mode: dockerfile
    description: "app-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/app-service-xxx.git
      branch: test #仓库扫描的分支,扫描这个仓库的test分支。
      sonarFilePath: release/docker/SonarDockerfile 
  - name: app-mobile
    mode: dockerfile
    description: "app-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/app-mobile-xxx.git
      branch: fix-bug #仓库扫描的分支,扫描这个仓库的fix-bug分支。
      sonarFilePath: release/docker/SonarDockerfile
```
### 样例三：扫描多组，这里组可以理解一个单元。
样例文件：[`multipleGroup`](http://172.17.189.70/sonar/insight-chart/-/blob/develop/examples/multipleGroup.yaml)
- 组的作用主要是区分不同的业务和不同的团队之间的项目，另外两个作用，
- 第一个是扫描优先级的问题，哪个组需要最开始扫描。
- 第二个作用是扫描效率问题，为了不影响其他group下的项目扫描时间，有一些项目，可能需要比较长的时间才能扫描完，如果同时扫描多个这种项目的话，那么对服务器的资源消耗比较大，所以比较大的项目，建议放在一组里面，使用parallel参数控制下。
``` yaml
global:
  sonar:
    dockerFilePath: release/docker/SonarDockerfile
    nativeFilePath: sonar-project.properties
    mode: dockerfile
  parallel: 5 #并行数，每一组group的项目一次扫描多少个项目，如果group没有指定parallel参数的话，那么会默认使用这个并行数。
  triggerTimeCron: "0 0 6 ? * MON" #任务扫描时间,每周一上午6点开始扫描
  repo:
    branch: develop #扫描的仓库分支，如果projects没有指定branch参数的话，那么会默认使用这个分支。
groups: #组定义，可以定义多个组，放在最前面的组，最先开始扫描。
- name: app-xxx #组名，可以重复，可以简单的定义为比如xxx业务下所有的前端服务，用英文表示
  parallel: 4 #并行数，一次扫描，扫描多少个项目。
  description: "app-xxx example groups,this is for testing!" #组描述
  projects: # 项目定义
  - name: app-web #项目名，可以重复
    mode: dockerfile #项目扫描方式，默认使用dockerfile
    description: "app-web is front-end project！"  #项目描述
    repo: #仓库定义
      url: git@172.17.189.70:xxxxx-groups/app-xxx.git #仓库地址，只支持ssh方式。
      branch: develop #仓库扫描的分支
      sonarFilePath: release/docker/SonarDockerfile #dockerfile 扫描的文件路径，这个路径是任务仓库里面的路径。
  - name: app-service
    mode: dockerfile
    description: "app-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/app-service-xxx.git
      branch: test #仓库扫描的分支,扫描这个仓库的test分支。
      sonarFilePath: release/docker/SonarDockerfile 
  - name: app-mobile
    mode: dockerfile
    description: "app-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/app-mobile-xxx.git
      branch: fix-bug #仓库扫描的分支,扫描这个仓库的fix-bug分支。
      sonarFilePath: release/docker/SonarDockerfile
- name: service-xxx  #第二组
  parallel: 4 
  description: "service-xxx example groups,this is for testing!"
  projects: 
  - name: service-web 
    mode: dockerfile 
    description: "service-web is front-end project！" 
    repo:
      url: git@172.17.189.70:xxxxx-groups/service-xxx.git 
      branch: develop 
      sonarFilePath: release/docker/SonarDockerfile
  - name: service-service
    mode: dockerfile
    description: "service-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/service-service-xxx.git
      branch: test 
      sonarFilePath: release/docker/SonarDockerfile 
  - name: service-mobile
    mode: dockerfile
    description: "service-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/service-mobile-xxx.git
      branch: fix-bug
      sonarFilePath: release/docker/SonarDockerfile
- name: platform-xxx #第三组
  parallel: 4 
  description: "platform-xxx example groups,this is for testing!"
  projects: 
  - name: platform-web 
    mode: dockerfile 
    description: "platform-web is front-end project！" 
    repo:
      url: git@172.17.189.70:xxxxx-groups/platform-xxx.git 
      branch: develop 
      sonarFilePath: release/docker/SonarDockerfile
  - name: platform-service
    mode: dockerfile
    description: "platform-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/platform-service-xxx.git
      branch: test 
      sonarFilePath: release/docker/SonarDockerfile 
  - name: platform-mobile
    mode: dockerfile
    description: "platform-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/platform-mobile-xxx.git
      branch: fix-bug
      sonarFilePath: release/docker/SonarDockerfile
```
### 样例四：使用全局配置，全部扫描统一分支、扫描方式。
``` yaml
global:
  sonar:
    dockerFilePath: release/docker/SonarDockerfile
    nativeFilePath: sonar-project.properties
    mode: dockerfile
  parallel: 3 #并行数，每一组group的项目一次扫描多少个项目，如果group没有指定parallel参数的话，那么会默认使用这个并行数。
  triggerTimeCron: "0 0 6 ? * MON" #任务扫描时间,每周一上午6点开始扫描
  repo:
    branch: develop #扫描的仓库分支，如果projects没有指定branch参数的话，那么会默认使用这个分支。
groups: #组定义，可以定义多个组，放在最前面的组，最先开始扫描。
- name: app-xxx #组名，可以重复，可以简单的定义为比如xxx业务下所有的前端服务，用英文表示
  description: "app-xxx example groups,this is for testing!" #组描述
  projects: # 项目定义
  - name: app-web #项目名，可以重复
    description: "app-web is front-end project！"  #项目描述
    repo: #仓库定义
      url: git@172.17.189.70:xxxxx-groups/app-xxx.git #仓库地址，只支持ssh方式。
  - name: app-service
    description: "app-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/app-service-xxx.git
  - name: app-mobile
    description: "app-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/app-mobile-xxx.git
- name: service-xxx  #第二组
  description: "service-xxx example groups,this is for testing!"
  projects: 
  - name: service-web 
    description: "service-web is front-end project！" 
    repo:
      url: git@172.17.189.70:xxxxx-groups/service-xxx.git 
  - name: service-service
    description: "service-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/service-service-xxx.git
  - name: service-mobile
    description: "service-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/service-mobile-xxx.git
- name: platform-xxx #第三组
  description: "platform-xxx example groups,this is for testing!"
  projects: 
  - name: platform-web 
    description: "platform-web is front-end project！" 
    repo:
      url: git@172.17.189.70:xxxxx-groups/platform-xxx.git 
  - name: platform-service
    description: "platform-service is back-end project！"  
    repo:
      url: git@172.17.189.70:xxxxx-groups/platform-service-xxx.git
  - name: platform-mobile
    description: "platform-mobile is mobile project！"
    repo: 
      url: git@172.17.189.70:xxxxx-groups/platform-mobile-xxx.git
```

# FAQ
#### 1. triggerTimeCron（扫描时间）不会定义怎么办？
在线生成地址： 
- [`pppet`](https://www.pppet.net/)
- [`cron-ciding`](http://cron.ciding.cc/)

#### 2. 常见问题

1. 扫描结果失败
##### 1. 错误消息：**java.lang.RuntimeException: java.lang.Exception: xxxx/SonarDockerfile file not exist!**
- 该项目xxxx/SonarDockerfile文件不存在， 如果在其他的目录，在扫描文件里面sonarFilePath字段填写下，如果不存在操作， 请创建并配置下这个扫描SonarDockerfile。  
##### 2. 错误消息：**java.lang.Exception: git clone fail**
- 克隆仓库失败，检查下这个项目是否存在，和分支。
- 检查调度组件的克隆私钥是否能被正常使用，[`code-analyzer`](http://172.17.189.70/sonar/code-analyzer)

#### 3. SonarDockerfile文件解释，请参考[`SonarDockerfile文档`](http://172.17.189.70/sonar/code-analyzer#%E4%BA%8Csonardockerfile%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89)
