---
title: CentOS 7 学习笔记
date: 2017-11-21 20:47:26
keywords: centos安装配置, yum命令, git命令, docker命令
categories: 笔记
tags:
- linux
- centos
- git
- docker
---
## network

### 配置
``` bash
nmtui 带界面的网络管理器
hostnamectl set-hostname [hostname] 设置主机名
vi /etc/sysconfig/network-scripts/ifcfg-eth0 编辑网卡配置
ONBOOT=yes #系统启动时激活
DNS1=223.5.5.5 #阿里DNS 
DNS2=223.6.6.6
systemctl restart network.service 重启网络服务
```

### 常用命令
``` bash
ip addr 查看IP信息
ss -tlnp 查看端口信息
ping 网络连通性测试
```

## yum

### 配置阿里镜像
``` bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 常用命令
``` bash
yum makecache 重建缓存
yum check-update 检查可更新的包
yum upgrade 更新所有包但不改变设置
yum update 更新所有包与系统内核
yum update [package] 更新指定包
yum list 显示已安装的包与可安装的包
yum list [package] 显示指定包的安装情况
yum search [package] 查询匹配的包
yum info [package] 显示指定包的详情信息
yum deplist [package] 显示指定包的依赖情况
yum install [package] 安装指定包
yum remove [package] 删除指定包
yum grouplist 显示已安装和可安装的组
yum groupinfo [group] 显示指定组的所有包
yum groupinstall [group] 安装指定组的所有包
yum groupremove [group] 删除指定组的所有包
```

## ssh

### 修改端口
``` bash
yum install policycoreutils-python
semanage port -a -t ssh_port_t -p tcp 10202
semanage port -l | grep ssh
vim /etc/ssh/sshd_config
Port 10202
```

## selinux

### 禁用
``` bash
vi /etc/selinux/config
SELINUX=disabled
```

## kernel

### 升级
``` bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
yum --enablerepo="elrepo-kernel" install kernel-ml
rpm -qa | grep kernel
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
```

### 开启TCP BBR
``` bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

## git

### 安装
``` bash
yum groupinstall "Development Tools"
yum install wget vim-enhanced asciidoc xmlto openssl-devel libcurl-devel zlib-devel perl-ExtUtils-MakeMaker
wget https://github.com/git/git/archive/v2.15.0.tar.gz
tar -zxvf v2.15.0.tar.gz
cd git-2.15.0
make configure
./configure --prefix=/usr/local
make && make install
```

### 服务端配置
``` bash
useradd git
passwd -d git
cd /home/git
git init --bare project.git
mkdir .ssh
touch .ssh/authorized_keys
chown -R git:git .ssh/
chown -R git:git project.git/
chmod 700 .ssh/
chmod 600 .ssh/authorized_keys
```

### 客户端配置
``` bash
ssh-keygen -t rsa
ssh root@centos 'cat >> /home/git/.ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```

### 常用命令
``` bash
git init 初始化新仓库
git add [file] 添加文件到暂存区
git reset HEAD [file] 删除文件从暂存区
git checkout -- [file] 丢弃文件在工作目录的改动
git status -v [file] 显示文件改动状态
git diff [file] 显示文件改动对比
git commit -m "msg" 提交暂存区所有修改到当前分支
git log --pretty=oneline 显示提交日志
git reflog 显示操作日志
git reset --hard [commit] 复位工作目录与索引
git clone [url] 克隆远程仓库到本地
git remote 显示所有远程仓库
git fetch 获取远程仓库更新
git push origin master 将分支推送到远程仓库
git checkout -b [branch] 创建并切换到新分支
git checkout [branch] 切换分支
git merge [branch] 合并指定分支到当前分支
git branch -v 显示本地分支
git branch -r 显示远程分支
git branch -d [branch] 删除分支
```

## shadowsocks

### 安装
``` bash
yum install python-setuptools && easy_install pip
pip install shadowsocks
vim /etc/shadowsocks.json
{
     "server": "0.0.0.0",
     "server_port": 443,
     "local_port": 1080,
     "password": "",
     "timeout": 600,
     "method": "aes-256-cfb"
}
vim /etc/systemd/system/shadowsocks.service
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
systemctl enable shadowsocks.service
systemctl start shadowsocks.service
```

### 科学上网
``` bash
wget https://github.com/rofl0r/proxychains-ng/releases/download/v4.12/proxychains-ng-4.12.tar.xz
tar -xvf proxychains-ng-4.12.tar.xz
cd proxychains-ng-4.12
./configure
make && make install && make install-config
vim /usr/local/etc/proxychains.conf
[ProxyList]
socks5 127.0.0.1 1080
vim ~/.bashrc
alias fq='proxychains4'
fq curl -I https://twitter.com
```

## java

### jdk
``` bash
wget http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz?AuthParam=1511329565_c70e7b46c6e4d05e46de8b5211234236
mkdir /usr/local/java
tar -zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/java
chown root:root -R /usr/local/java
vim ~/.bash_profile
JAVA_HOME=/usr/local/java/jdk1.8.0_151
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH
java -version
```

### maven
``` bash
wget https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
mkdir /usr/local/apache
tar -zxvf apache-maven-3.5.2-bin.tar.gz -C /usr/local/apache
chown root:root -R /usr/local/apache
vim ~/.bash_profile
MAVEN_HOME=/usr/local/apache/apache-maven-3.5.2
PATH=$PATH:$HOME/bin:$MAVEN_HOME/bin
export MAVEN_HOME PATH
mvn -version
```

### nexus
``` bash
wget http://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz -C /opt
mv /opt/nexus-3.6.1-02 /opt/nexus
useradd nexus
chown nexus:nexus -R /opt/nexus
chown nexus:nexus -R /opt/sonatype-work
vim /opt/nexus/bin/nexus.rc
run_as_user="nexus"
vim /etc/systemd/system/nexus.service
[Unit]
Description=nexus service
After=network.target
[Service]
Type=forking
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
[Install]
WantedBy=multi-user.target
systemctl enable nexus
systemctl start nexus
firewall-cmd --zone=public --permanent --add-port=8081/tcp
firewall-cmd --reload
```

## nodejs

### 安装
``` bash
curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -
yum -y install nodejs
node --version
```

### nrm
``` bash
npm install -g nrm
nrm use taobao
```

## nginx

### 安装
``` bash
wget https://nginx.org/download/nginx-1.13.7.tar.gz
tar -zxvf nginx-1.13.7.tar.gz
cd nginx-1.13.7
./configure --prefix=/usr/local --with-http_ssl_module
make && make install
vim /usr/local/conf/nginx.conf
```

### 常用命令
``` bash
nginx 启动
nginx -s reload 重启
nginx -s stop 停止
```

## docker

### 安装
``` bash
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast
yum install -y docker-ce
vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --registry-mirror=http://f0e7f1ce.m.daocloud.io
systemctl enable docker.service
systemctl start docker.service
```

### 常用命令
``` bash
dockr version 查看版本信息
docker info 查看系统信息
docker search [image] 搜索镜像
docker pull [image] 拉取镜像
docker push [image|repository]:TAG 推送镜像
docker rmi [image...] 删除一个或多个镜像
docker rmi $(docker images -q -f dangling=true) 删除所有虚悬镜像
docker inspect [image|container] 查看镜像或容器的底层信息
docker images -a 列出所有镜像
docker ps -a 显示所有容器
docker stats -a 显示所有容器使用情况
docker logs [container] 查看容器日志
docker rm [container...] 删除一个或多个容器
docker rm $(docker ps -a -q) 删除所有容器
docker start/stop/restart [container] 启动/停止/重启容器
docker attach [container] 附加到运行中的容器
docker exec -it [container] bash 交互式进入到运行中的容器
docker run -it -p [host_port:contain_port] 将容器的端口映射到宿主机的端口
docker port [container] 查看容器端口映射信息
docker diff [container] 查看容器改动信息
docker commit [container] [repo:tag] 将一个容器固化为一个新的镜像
```

### docker-compose
``` bash
pip install docker-compose
```

### mysql
``` bash
docker pull mysql:latest
docker run --name mysqldb -v /var/mysqldb/my.cnf:/etc/mysql/my.cnf -v /var/mysqldb/data:/var/lib/mysql -p 0.0.0.0:3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:latest
docker cp /etc/localtime mysqldb:/etc/localtime
vim /var/mysqldb/my.cnf
[mysqld]
innodb_file_per_table=1
innodb_file_format=Barracuda
innodb_large_prefix=1
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
wait_timeout=31536000
interactive_timeout=31536000
docker restart mysqldb
```
