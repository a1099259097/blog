---
title: 使用 Travis CI 自动部署 Hexo 博客
date: 2017-11-23 10:46:10
keywords: hexo持续集成, travis-ci自动部署
categories: 笔记
tags:
- hexo
- travis-ci
---
## 想法
由于自己的博客部署在vps，每次更新都要手动打包上传，并且还依赖nodejs hexo-cli环境，换一台电脑就更麻烦了，于是想到了持续集成(CI)，因为博客源码托管在github，所以选择使用travis来做持续集成部署，如果私有项目建议使用jenkins

## 需求
在任何电脑上，只需装个git，pull源码，编辑md，push到github，触发travis ci，自动发版到vps，是不是很爽？！

## 实现

### 流程
1. 博客提交修改后push到github
2. github通知travis ci项目需要构建
3. travis ci立马安排构建
4. 构建完成后将结果push到vps
5. vps利用git钩子将结果部署到web容器

### 准备工作
1. 登陆[Travis CI](https://travis-ci.org), 同步账户并打勾你的博客项目
2. 进入项目配置页面，打勾`Build only if .travis.yml is present`
3. 博客项目根目录中创建 .travis.yml 文件
4. 生成访问vps的ssh-key
`ssh-keygen -t rsa -C "youremail@your.com” -f ~/.ssh/vps_rsa`
5. 将公钥写入vps authorized_keys
`ssh git@vps 'cat >> ~/.ssh/authorized_keys' < ~/.ssh/vps_rsa.pub`
6. 加密私钥，防止被利用你懂的
``` bash
# 安装ruby
curl -sSL https://get.rvm.io | bash -s stable --ruby
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
# 安装travis cli
gem install travis
# 登陆travis
travis login --auto
# 加密私钥生成vps_rsa.enc并创建用来解密的token会自动写入.travis.yml
travis encrypt-file ~/.ssh/vps_rsa --add -r github_user/blog_project
```

### 编写travis脚本
``` yaml
language: node_js
node_js: stable
branchs:
  only:
  - master
cache:
  directories:
  - node_modules
before_install:
# 解密私钥
- openssl aes-256-cbc -K $encrypted_***_key -iv $encrypted_***_iv -in vps_rsa.enc -out ~/.ssh/vps_rsa -d
# 修改权限
- chmod 600 ~/.ssh/vps_rsa
# 将私钥添加到缓存
- eval $(ssh-agent)
- ssh-add ~/.ssh/vps_rsa
# 生成ssh config 在travis后台配置 $vps_host $vps_port 两个环境变量
- echo -e "Host vps\n\tHostName $vps_host\n\tPort $vps_port\n\tUser git\n\tStrictHostKeyChecking no\n\tIdentityFile ~/.ssh/vps_rsa\n\tIdentitiesOnly yes" >> ~/.ssh/config
# 安装hexo-cli
- npm install -g hexo-cli
install:
# 安装依赖包
- npm install
script:
# 生成静态文件
- hexo clean
- hexo generate
after_success:
# push到vps仓库
- cd ./public
- git init
- git config user.name "shawnho"
- git config user.email "hi@shawnho.me"
- git add .
- git commit -m "deploy blog"
- git push --force --quiet "git@vps:blog.git" master:master
```

### 编写git钩子脚本
1. vps上创建blog.git仓库
``` bash
ssh vps
git init --bare /home/git/blog.git
cd /home/git/blog.git/hooks
vim post-receive
chmod +x post-receive
```
2. post-receive
``` bash
#!/bin/bash
git clone /home/git/blog.git /tmp/blog
rm -rf /var/www/blog/*
mv /tmp/blog/* /var/www/blog
rm -rf /tmp/blog
```
3. 配置完成，修改博客后push到github，travis ci将自动部署，爽吧！
