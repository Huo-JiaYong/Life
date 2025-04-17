### **http://docs.idevops.site/ 教程笔记**



## GitLab CICD

优点：

1. 使用 gitlab 代码仓库进行集成，没有额外的运营成本
2. 自动化程度高，提交了代码自动执行（也可以看作缺点）

### 基础概念

​	pipeline：执行一次流程任务

​	stage：一次任务可以划分为多个阶段

​	job：一个阶段可以划分为多个 job



### 基础语法

```yaml
# 前置脚本
before_script:
	- echo "hello, CICD"

# 流程列表（顺序执行）
stages:
	- info
	- clean
	- build
	- install

# 全局默认流程： .pre（前置） .post（后置）

# 1.JOB 名称（stage 下多个 job）
java-info:
	# 对应 stages 中的名称
	stage: info
	# 对应 runner 的 tags
	tags: java-runner
	# 主要执行的脚本语句
	script: 
		- java -version
	before_script: 
		- "xxxx"
	after_script:
		- "xxxx"
	
	# 2.指定 JOB 参数
	# 可以失败
	allow_failure: true
	# 控制作业的是否运行 全部成功 / 全部失败 / 一直 / 手动 / 延迟
	when: on_success / on_failure / always / manual / delayed 
	# 延迟时间
	start_in: “30”
	# 重试次数：[0~2]
	retry: 2
	# 超时时间，和项目时间谁小用谁
	timeout: 1h 1m 1s
	# 创建数量的相同 JOB 执行 (显示: 1/n ~ n/n)
	parallel: 5
	
	# 指定分支（逐渐不用）
	only: master
	# 排除分支
	except: master
	# 指定分支的选择规则（不能同 only / except 组合使用）
	rules:
		- if: '$DOMAIN == example.com || $DOMAIN == huo.com'
			when: manual
		- when: on_success	
# 指定整个管道的创建
workflow:
	rules:
		- if: 'xxxx'
		
# 3.全局缓存：每个阶段都要进行执行缓存的下载和上传操作
cache:
	key:
		# 指定文件发生了变化才缓存（最多2个）
		files:
			- file-path1
			- file-path2
	paths: 
		- path1
		- path2
		
# 4.将指定的文件，搜集到 gitlab
JOB:
    artifacts:
        name: ${poject-name}
        when: on_success / on_failure / always
        path:
            - target/*.jar
        reports: 
            juint: target/surefire-reports/TEST-*.xml
        # 文件在 gitlab 上的过期时间
        expire_in: 1yrs1mos1day1h2min / 1 years and 2 months and 1 weeks and 2 days
    # 指定某个 job 产生的制品
	dependencies:
		- ${JOB}
		
# 5.无序的执行作业
BUILD-1:
	stage: build
	script: xxx
BUILD-2:
	stage: build
	script: xxx
	
DEPLOY-1:
	stage: deploy
	script: xxx
	# 指定 BUILD-1 完成即可开始 deploy 阶段的 DEPLOY-1 作业（而不用等待 build 阶段都完成）
	needs: ["BUILD-1"]
	
# 6.include 引入 gitlab 文件 / extends 继承
include:
	local: 'ci/module-ci.yml'
	# 其他项目配置
	- project: gitlab-group/project-name
	  ref: master
	  file: '.gitlab-ci.yml'
	# 引用远程地址
	- remote: 'https://gitlab.com/awesome-project/raw/master/.gitlab-ci-template.yml'
	# 引入官方的模板 https://gitlab.com/gitlab-org/gitlab/tree/master/lib/gitlab/ci/templates
	- template: Maven.gitlab-ci.yml
# 指定父级 job，参数合并，自己优先
JOB:
	extends: .PARENT_JOB
# triger：触发其他的 pipeline
JOB:
	trigger:
		project: gitlab-group/project-name
		branch: master
		# 本项目：父子管道
		include: ci/module-ci.yml
		strategy: depend
		
# 7.使用 docker 方式（需要指定对应的 runner 的 executor "docker" 模式）
image: maven:3.6.3-jdk-8
# 指定 job 的镜像 
JOB:
	image: maven:3.6.3-jdk-8
# 指定依赖的 docker 容器
services:
	# image 名称
	- name: mysql:latest
	  alias: mysql-1
# 指定环境（如部署）
JOB:
	environment:
		name: prod
		url: http://localhost

# 禁用默认变量 / 环境变量
inherit:
	default: false
	variables: false
	
# 前置脚本
after_script:
	- echo "Bye, CICD"
```



### 安装

CentOS 7 安装流程 https://zhuanlan.zhihu.com/p/534072989 

1. 下载基础依赖 yum install policycoreutils openssh-server openssh-client postfix
2. 使用清华镜像站下载 rpm 包
3. 使用 rpm -ivh gitlab-ce-15.0.0-ce.0.el7.x86_64.rpm 安装

OR

1. 创建：/etc/yum.repos.d/gitlab-ce.repo

   ```
   [gitlab-ce]
   name=gitlab-ce
   baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
   Repo_gpgcheck=0
   Enabled=1
   Gpgkey=https://packages.gitlab.com/gpg.key
   ```

2. yum install gitlab-ce 安装最新版

##### 启动失败：

```shell
sudo systemctl start gitlab-runsvdir

sudo gitlab-ctl start
```



### Gitlab-Runner

#### Windows 

为了使用 maven 和 java 方便，所以直接在 windows 上使用（Windows会增加难度~）

下载和 gitlab 版本对应的 runner ：

https://s3.amazonaws.com/gitlab-runner-downloads/v16.10.0/binaries/gitlab-runner-windows-amd64.exe

可以直接修改 v16.10.0 版本号获取其他版本（不用代理下载的快点~）

##### 注册节点

1. gitlab-runner.exe install
2. gitlab-runner.exe start
3. gitlab-runner.exe register --url "xxxx" --token "xxxx"

##### 执行任务

1. 默认使用 pwsh 也就是 powershell as admin，但是执行会说权限不足
2. 修改 config.toml 中的 shell 属性为 cmd （cmd 执行命令的结果可能不正确，可能需要将命令写在一行）



### CICD 脚本



## Jenkins

优点：

1. 可以指定拉取仓库，gitlab、github 等
2. 生态支持更好，插件更丰富