
<p align="center">
<img width="130" align="center" src="img/logo.svg"/>
</p>
<h1 align="center">Docker Jenkins 安装</h1>



Jenkins 是一款能提高效率的软件，它能帮你把软件开发过程形成工作流，典型的工作流包括以下几个步骤：开发、提交、编译、测试、发布。

- Docker 的局限性之一，它只能用在 64 位的操作系统上。
- Jenkins 大概流程：从git上获取代码(或者钩子触发)、通过DockerFile构建镜像、推送镜像到远程仓库、服务器从远程仓库获取镜像、服务器停止并删除旧版本容器、run或者通过docker-compose运行新容器。


# **[1.安装 Jenkins](https://jenkins.io/doc/pipeline/tour/getting-started/)**

~~~
# 以CentOS为例
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum -y install jenkins

# java版本更新(针对gij版本)
$ java -version
java version "1.5.0"
gij (GNU libgcj) version 4.4.6 20110731 (Red Hat 4.4.6-3)
$ sudo yum remove java
$ sudo yum install -y java-1.7.0-openjdk
$ java -version
java version "1.7.0_79"
OpenJDK Runtime Environment (rhel-2.5.5.1.el6_6-x86_64 u79-b14)
OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)

# 如果你的Jenkins使用git作为数据传输的管道，那么的所有Jenkins节点都要安装git
$ sudo yum install -y git

# 设置git账户
$ git config --global user.name "yourname"
$ git config --global user.email "yourmail"

# 安装成功后，配置文件在/etc/sysconfig/jenkins下，默认端口为8080，你需要设置一下防火墙，让该端口可以被外部访问
# 设置开机启动
$ sudo chkconfig jenkins on # 确保系统中有一个jenkins账户，如果没有则需要创建，理论上安装了Jenkins后，会自动创建该用户

# 然后创建ssh密钥，密钥被用来在多个节点中进行免密访问，同时帮助打通git数据通道。
# 在这之前要确认jenkins用户的home目录是否有效（在下面的例子中home是/var/bin/jenkins），并切换到jenkins用户下
$ grep jenkins /etc/passwd
jenkins:x:496:496:Jenkins Continuous Integration Server:/var/lib/jenkins:/bin/bash
$ su jenkins
$ cd ~
$ pwd
/var/lib/jenkins

# 创建非对称密钥，执行ssh-keygen命令，并一路回车
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/fengyajie/.ssh/id_rsa): Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/fengyajie/.ssh/id_rsa.
Your public key has been saved in /home/fengyajie/.ssh/id_rsa.pub.
The key fingerprint is:
The key's randomart image is:
+--[ RSA 2048]----+
|        ....   +=|
|        ... .....|
|         . ...o +|
|          E. . *.|
|        S  .= +  |
|         . o + . |
|          . o o  |
|             o o |
|              o  |
+-----------------+
$ ls ~/.ssh/
id_rsa  id_rsa.pub  known_hosts

# Jenkins是一个Master-Slave的架构，它可以把任务发布到不同的节点上执行，典型的应用场景是你有2个运行环境，
# 一个是测试环境，一个是生产环境，你可以指定工作流中，哪些任务在测试环境中执行，哪些任务在生产环境中执行。
# 如果你需要配置Slave，在Slave节点上创建一个jenkins用户，并建立Master和Slave的授信关系
#（你需要将下面的host替换为具体的服务器IP，注意一定要保证Master和Slave之间是内网通信的，否则公网环境延迟较大）
ssh jenkins@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

# 同时，为了让jenkins可以执行更高权限的命令，所有节点都需要把jenkins用户设置为sudo用户。
# 当然，我这是为了偷懒，更好的办法是设置一个专门的用户组，让这个组有一定的权限，然后把jenkins加入到这个用户组。
$ sudo grep jenkins /etc/sudoers
jenkins   ALL=(ALL)   NOPASSWD: ALL

~~~

# **[2.设置Slave、创建Pipeline](https://www.jianshu.com/p/b524b151d35f)**

# **[3.ASP.NET Core Jenkins Docker 实现一键化部署](https://mp.weixin.qq.com/s?__biz=MzAxMTMxMDQ3Mw==&mid=2660103306&idx=1&sn=01f8743eceb9092590b363c6ab6f4fac&chksm=803a506cb74dd97a7605503cad5c2cf049eb402f845998b39dd7ba13043a78021fd5db015082&mpshare=1&scene=23&srcid=11269lOP7TG0BhlzfkfHO1XU#rd)
