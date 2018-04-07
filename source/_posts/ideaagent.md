---
title: IntelliJ IDEA 辅助工具 ideaagent
date: 2017-12-20 19:14:16
keywords: mybatis plugin crack, iedis crack
categories: 笔记
tags:
- java
- idea
- crack
---
> IntelliJ IDEA 是本人常用的IDE之一，其中有两款插件特别好用，[Mybatis plugin 3.58](https://plugins.jetbrains.com/plugin/7293-mybatis-plugin) 和 [Iedis 2.56](https://plugins.jetbrains.com/plugin/9228-iedis)，由于不是免费的，所以制作了本工具来绕过限制。

ideaagent v1.2 下载地址 [https://github.com/mrshawnho/ideaagent](https://github.com/mrshawnho/ideaagent)
更新于 2018-04-07

## 特色功能
### ideaagent v1.2
Mybatis plugin v3.58 crack
Iedis v2.56 crack
支持 IntelliJ IDEA 2018.1

### ideaagent v1.12
Mybatis plugin v3.53 crack
Iedis v2.43 crack
支持 IntelliJ IDEA 2017.3.5

## 使用方法
1. 下载 [ideaagent-1.2.jar](https://github.com/mrshawnho/ideaagent/releases)
2. 打开 idea.vmoptions (Help -> Edit Custom VM Options...)
最下方插入 `-javaagent:/download/ideaagent-1.2.jar`
![](/uploads/ideaagent/001.png)
3. 重启 idea
首次启动需要信任本地服务器 ssl 证书，点击接受后如未激活，再次重启即可
![](/uploads/ideaagent/002.png)

## 测试环境
1. Windows 10 && MacOS High Sierra
2. JDK 1.8
3. IDEA 2018.1

## 技术实现
1. 使用 java instrumentation 创建代理
2. 使用 javassist 动态修改代码
3. 使用 vertx web 创建本地认证服务器

本文为个人学习笔记，切勿用于非法用途，转载请注明出处。
