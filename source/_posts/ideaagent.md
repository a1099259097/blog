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
> IntelliJ IDEA 是本人常用的IDE之一，其中有两款插件特别好用，[Mybatis plugin](https://plugins.jetbrains.com/plugin/7293-mybatis-plugin) 和 [Iedis](https://plugins.jetbrains.com/plugin/9228-iedis)，由于不是免费的，所以制作了本工具来绕过限制。

ideaagent 下载 [https://github.com/mrshawnho/ideaagent](https://github.com/mrshawnho/ideaagent)

## 插件分析

### Mybatis plugin v3.42
该版使用 Kotlin 开发，代码做了混淆，更变态的是将核心代码藏在了 mybatis-generator-core-1.3.5.jar
包中的 DeleteByPrimaryKeyElementGenerator.class 文件中以供插件运行时动态加载，对修改插件字节码而达到目的增加了一定难度。

继续分析后发现插件是通过服务端请求认证的，那么我们修改为本地认证就好啦，最初以为改改hosts文件伪造请求就可以了，但没那么简单，内容通过了RSA加密与验签，没有私钥是徒劳的，那么暴力点把公钥改了，皆大欢喜！

### Iedis v2.43
该版只是做了代码混淆，同上的思路也能达到目的。

## 技术实现
1. 通过 java instrumentation 技术创建 IDEA 的代理程序
2. 使用 javassist 动态修改代码
3. 使用 vert.x 创建本地 http 认证服务器

## 使用方法
1. 添加 VM option (IDEA Help > Edit Custom VM Options...)
`-javaagent:ideaagent-1.0-jar-with-dependencies.jar`
2. 重启 IDEA

本文为个人学习笔记，切勿用于非法用途，转载请注明出处。
