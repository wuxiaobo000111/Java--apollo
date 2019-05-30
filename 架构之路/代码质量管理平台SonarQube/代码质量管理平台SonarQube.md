> https://www.cnblogs.com/qiumingcheng/p/7253917.html


# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;SonarQube是管理代码质量一个开放平台，可以快速的定位代码中潜在的或者明显的错误，下面将会介绍一下这个工具的安装、配置以及使用。准备工作；

```text
1、jdk（不再介绍）

2、sonarqube：http://www.sonarqube.org/downloads/

3、SonarQube+Scanner:https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-2.5.zip

4、mysql数据库（不再介绍）
```

# 安装篇

>&nbsp;&nbsp;&nbsp;&nbsp;1.下载好sonarqube后，解压打开bin目录，启动相应OS目录下的StartSonar。如本文演示使用的是win的64位系统，则打开D:\sonar\sonarqube-5.3\sonarqube-5.3\bin\windows-x86-64\StartSonar.bat
>&nbsp;&nbsp;&nbsp;&nbsp;启动浏览器，访问http://localhost:9000，如出现下图则表示安装成功。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/47.png?raw=true)


# 配置篇

>&nbsp;&nbsp;&nbsp;&nbsp;打开mysql，新建一个数据库。

>&nbsp;&nbsp;&nbsp;&nbsp;打开sonarqube安装目录下的D:\sonar\sonarqube-5.3\sonarqube-5.3\conf\sonar.properties文件

>&nbsp;&nbsp;&nbsp;&nbsp;在mysql5.X节点下输入以下信息

```properties
sonar.jdbc.url=jdbc:mysql://172.16.30.228:3306/qjfsonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.jdbc.username=gmsd
sonar.jdbc.password=gmsdtrade
sonar.sorceEncoding=UTF-8
sonar.login=admin
sonar.password=admin
<!-- url是数据库连接地址，username是数据库用户名，jdbc.password是数据库密码，
login是sonarqube的登录名，sonar.password是sonarqube的密码 -->
```

>&nbsp;&nbsp;&nbsp;&nbsp;重启sonarqube服务，再次访问http://localhost:9000，会稍微有点慢，因为要初始化数据库信息

>&nbsp;&nbsp;&nbsp;&nbsp;数据库初始化成功后，登录

>&nbsp;&nbsp;&nbsp;&nbsp;按照下图的点击顺序，进入插件安装页面

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/48.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;7.搜索chinese Pack，安装中文语言包

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/49.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;安装成功后，重启sonarqube服务，再次访问http://localhost:9000/，即可看到中文界面

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/50.png?raw=true)


# 使用篇

>&nbsp;&nbsp;&nbsp;&nbsp;打开D:\sonar\sonar-scanner-2.5\conf\sonar-runner.properties文件

>&nbsp;&nbsp;&nbsp;&nbsp;mysql节点下输入以下信息

```properties
sonar.jdbc.url=jdbc:mysql://172.16.30.228:3306/qjfsonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.jdbc.username=gmsd
sonar.jdbc.password=gmsdtrade

注意：如果测试项目与服务器不在同一台机子，则需要添加服务器的IP：

#----- Default SonarQube server
sonar.host.url=http://XXX.XXX.XXX.XXX:9000
```
>&nbsp;&nbsp;&nbsp;&nbsp;配置环境变量

```text
a.新建变量，name=SONAR_RUNNER_HOME。value=D:\sonar\sonar-scanner-2.5

b.打开path，输入%SONAR_RUNNER_HOME%\bin;

c.sonar-runner -version，出现以下信息，则表示环境变量设置成功
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/51.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;4.打开要进行代码分析的项目根目录，新建sonar-project.properties文件

>&nbsp;&nbsp;&nbsp;&nbsp;5.输入以下信息

```text

# must be unique in a given SonarQube instance
sonar.projectKey=my:project
# this is the name displayed in the SonarQube UI
sonar.projectName=apiautocore
sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# Since SonarQube 4.2, this property is optional if sonar.modules is set. 
# If not set, SonarQube starts looking for source code from the directory containing 
# the sonar-project.properties file.
sonar.sources=src
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8


其中：projectName是项目名字，sources是源文件所在的目录
```

>&nbsp;&nbsp;&nbsp;&nbsp;6.设置成功后，启动sonarqube服务，并启动cmd

>&nbsp;&nbsp;&nbsp;&nbsp;7.在cmd进入项目所在的根目录，输入命令：sonar-runner，分析成功后会出现下图

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/52.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;打开http://localhost:9000/，我们会看到主页出现了分析项目的概要图

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/53.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;我们点击项目，选择问题链接，会看到分析代码的bug，哇，好多

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/54.png?raw=true)