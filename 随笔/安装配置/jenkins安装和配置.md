#### centos yum部署jenkins，2024年支持版本

> jenkins建议安装[LTS](https://so.csdn.net/so/search?q=LTS&spm=1001.2101.3001.7020)长期支持版本，而不是安装每周更新版本，[jenkins安装指定版本](https://mirrors.jenkins-ci.org/redhat/)
> [openjdk官网下载](https://adoptium.net/zh-CN/temurin/releases/?version=17&arch=x64&os=linux&package=jdk)
>
> [Index of /jenkins/redhat-stable/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/)
>
> Jenkins 宣布：Jenkins 2.357 以后要求jdk最低版本 11，否则启动会报错

#### 1、openjdk 17配置

- 下载jdk：https://adoptium.net/zh-CN/temurin/releases/?version=17&arch=x64&os=linux&package=jdk

- 获取文件后，解压移动

  [root@192 ~]# ls OpenJDK17U-jdk_x64_linux_hotspot_17.0.12_7.tar.gz
  OpenJDK17U-jdk_x64_linux_hotspot_17.0.12_7.tar.gz

  [root@192 ~]# tar xf OpenJDK17U-jdk_x64_linux_hotspot_17.0.12_7.tar.gz

  [root@192 ~]#mv jdk-17.0.12+7/ /usr/local/

  [root@192 ~]# ls /usr/local/jdk-17.0.12+7/
  bin conf include jmods legal lib man NOTICE release

- 绑定软链接，要不然会报错

  [root@192 ~]# ln -s /usr/local/jdk-17.0.12+7/bin/java /usr/bin/java

- 环境变量配置

  vi /etc/profile

  ```
  export JAVA_HOME=/usr/local/jdk-17.0.12+7
  export JRE_HOME=$JAVA_HOME/jre
  export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
  export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
  ```

  source /etc/profile

- openjdk 17 安装完成

  [root@192 ~]# java -version
  openjdk version “17.0.12” 2024-07-16
  OpenJDK Runtime Environment Temurin-17.0.12+7 (build 17.0.12+7)
  OpenJDK 64-Bit Server VM Temurin-17.0.12+7 (build 17.0.12+7, mixed mode, sharing)
  [root@192 ~]#

- 2.480-1.1.noarch.rpm 安装完成

  获取文件后，安装

  [root@192 ~]# ls 2.480-1.1.noarch.rpm
  2.480-1.1.noarch.rpm

  [root@192 ~]# yum install -y 2.480-1.1.noarch.rpm

- jenkins配置
  vi /usr/lib/systemd/system/jenkins.service

  ```
  修改：
  	User=root 
  	Group=root 
  	Environment=“JAVA_HOME=/usr/local/jdk-17.0.12+7/”
      Environment=“JENKINS_JAVA_CMD=/usr/local/jdk-17.0.12+7/bin/java”
      8080访问端口是默认高危常见端口，可以自定义：
      Environment=“JENKINS_PORT=8080”
  
  
  # $JENKINS_WEBROOT.
  User=root
  Group=root
  
  # The Java home directory. When left empty, JENKINS_JAVA_CMD and PATH are consulted.
  #Environment="JAVA_HOME=/usr/local/jdk-17.0.13+11/"
  
  # The Java executable. When left empty, JAVA_HOME and PATH are consulted.
  #Environment="JENKINS_JAVA_CMD=/usr/local/jdk-17.0.13+11/bin/java"
  
  # Port to listen on for HTTP requests. Set to -1 to disable.
  # To be able to listen on privileged ports (port numbers less than 1024),
  # add the CAP_NET_BIND_SERVICE capability to the AmbientCapabilities
  # directive below.
  Environment="JENKINS_PORT=8089"
  ```

- 启动jenkins，并开机自启

  [root@192 ~]# systemctl start jenkins

  [root@192 ~]# systemctl status jenkins
  ● jenkins.service - Jenkins Continuous Integration Server
  Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; vendor preset: disabled)
  Active: active (running) since Wed 2024-11-20 17:19:51 CST; 2 days ago

  [root@192 ~]# ss -atnl
  LISTEN 0 50  *:8089 

  [root@192 ~]# systemctl enable jenkins

- 访问jenkins

  ip+8089 登录帐号/密码

- jenkins替换国内插件源

  > jenkins更换插件源的方法我试过了，发现清华镜像源的`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`里面的路径依然是国外的，替换了也没用。所以得更改这个文件里的连接。

  - 找到update-center.json文件
    不同安装方式的文件位置不一样，找一找就能找到，我是rpm包安装的，在`/var/lib/jenkins/updates/`下

    ```
    [root@192 ~]# cd /var/lib/jenkins/updates/
    [root@192 updates]# ls
    default.json                           hudson.tasks.Maven.MavenInstaller
    ```

  - 修改update-center.json文件不同版本要替换的连接不一样，我这里的是`https://updates.jenkins.io/download`,有的可能是`http://updates.jenkins-ci.org/download`
    都替换成`https://mirrors.tuna.tsinghua.edu.cn/jenkins`就行了

    ```
    sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' default.json && sed -i 's#http://www.google.com#https://www.baidu.com#g' default.json
    ```

  - 更改jenkins插件管理中的URL
    Dashboard > 插件管理 > 高级 > 升级站点 > URL
    URL更改为`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`
    重启之后安装插件就很快了

