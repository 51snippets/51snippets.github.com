---
layout: post
title: "完成Octopress环境的搭建"
date: 2013-01-20 10:38
comments: true
categories: 
---
#Octopress是神马神器？
[Octopress.org](http://octopress.org/docs)是一个基于Ruby的开源Blogging Framework，Octopress是基于[Jekyll]()，两者有所区别，
总之Octopress方便多了
<!-- more -->
#安装准备
之前本来想用Jekyll来利用github做blog，后面发现使用Octopress会更方便，就转而搭起Octopress，本人电脑是Mac系统

##1、创建github账号，像我的账号是51snipets

### 配置ssh登录github
```
[[ -f ~/.ssh/id_rsa.pub ]] || ssh-keygen -t rsa
```
这个是查看是否已经有rsa公钥，使用ssh方式登录github需要的东西，如果.ssh目录下不存在id_rsa.pub则生成ras公钥私钥，一路按照提示完成操作

```
pbcopy < ~/.ssh/id_rsa.pub
```
将公钥信息拷到剪贴板，到github个人账号的ssh设置页，增加公钥，将内容拷入保存，完成操作

### 增加博客repository
在github中增加username.github.com的repository，如：51snippets.github.com

##2、配置ruby环境

### 安装rvm，再使用rvm安装ruby 1.9.3以上版本（mac自带的是ruby1.8.7）
```
bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer) # 安装RVM
echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" ' >> ~/.bash_profile    # 添加环境变量到 ~/.bash_profile
source ~/.bash_profile  # source 一下，让它起作用
rvm install 1.9.3    # 安装ruby1.9.3
rvm 1.9.3 --default  # 设置ruby默认版本为1.9.3
ruby --version       # 查看当前ruby版本是否已经被设置1.9.3
```
### 安装Octopress
```
git clone git://github.com/imathis/octopress.git octopress  #从github clone octopress的源代码
cd octopress
gem install bundler
bundle install
rake install  # 安装主题
```
### 提交
```
rake setup_github_pages # 和github创建关联
git@github.com:your_username/your_username.github.com.git   # 提示输入github URL
rake generate # 把你所有编辑的内容生成你的Blog静态页面
rake deploy   # 如果检查没有任何问题就可以 push 你的 blog 到 github master branch
```
### 备份代码
```
git add .
git commit -m 'first commit'
git push origin source # 如果这一步出错，请再次检查仓库名称是否按要求命名，同时检查Admin面板下Default Branch是否为 master
```
### 使用
```
git pull octopress master     # Get the latest Octopress
bundle install                # Keep gems updated
rake update_source            # update the template's source
rake update_style             # update the template's style
```

