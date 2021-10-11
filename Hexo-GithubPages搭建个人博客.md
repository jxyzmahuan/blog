---
title: Hexo&GithubPages搭建个人博客
date: 2021-09-16 21:24:59
tags:
    - Hexo
    - GitHubPages
---

# Github配置

## 下载安装git

[点击下载Git](https://git-scm.com/download/win)

## 设置GitHub的user name和email

```
##命令行执行
git config --global user.name "jxyzmahuan" 
git config --global user.email "690040143@qq.com"
```

## 生成SSH密钥

打开 Git Bash，输入如下命令，然后连续按三个回车即可

```
#生成的key默认在C:\Users\用户名\.ssh下,id_rsa为私钥，id_rsa.pub为公钥
ssh-keygen -t rsa -C "690040143@qq.com"
```

## 将SSH公钥添加到GitHub账户

1. 登录GitHub，

2. 点击右上角settings--->SSH and GPG keys

3. 点击 New  SSH key，title任意起个名字，将id_rsa.pub复制到key文本框，点击Add SSH key

## 测试连接

```
#打开 Git Bash 输入：
ssh -T git@github.com

Hi jxyzmahuan! You've successfully authenticated, but GitHub does not provide shell access.
```



# Hexo安装配置

## 下载nodejs

windows系统下[nodejs下载](https://nodejs.org/dist/v14.17.6/node-v14.17.6-x64.msi)，安装nodejs

## 安装hexo

```
npm install -g hexo-cli
```

## 初始化hexo

```
hexo init blog
cd blog
npm install
```

## 启动hexo

```
hexo s
```

访问连接 http://localhost:4000

