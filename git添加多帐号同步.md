---
title: GIT添加GITHUB多帐号同步
tags:
- git
categories:
- Ronger
- git
- 多帐号
---
# 背景

电脑里已经有了一个全局的GITHUB帐号,匹配的RSA密钥等

# 需求
另外的一个项目需要一个新的GITHUB帐号,且多个项目(帐号)之间能够互不影响的PUSH和PULL

# 思路
删除GITHUB帐号的全局配置,改为按项目配置;生成新的RSA密钥并添加至GITHUB;增加'.ssh'下的RSA密钥**config**配置文件和GITHUB++主机别名++配置,且修改新增项目GITHUB主机的配置为别名配置

<!-- more -->

# 具体操作大纲
## 取消用户的***global***配置
```bash

cd ~/.ssh #切换到users\admin\.ssh
git config --global --unset user.name
git config --global --unset user.email
```
## 设置每个repo自己的user信息
```bash
git config user.name "kkaxiao"
git config user.emamil "kkaxiao@gmail.com"
```
##  新建GITHUB第二个帐号guanshuige的SSH KEY
```bash
#切换到.ssh
ssh-keygen -t rsa -C "guanshuige@gmail.com" -f id_rsa_guanshuige 
#新ssh key命名为id_rsa_guanshuige以区分id_rsa的默认命名
```
## 密钥添加到SSH AGENT当中
> 因为默认只读取id_rsa,为了让SSH识别新的私钥,需要将其添加到ssh agent中
```bash
$ ssh-add id_rsa_guanshuige
```
如果出现Could not open a connection to your authentication agent的错误,就用下面的这个命令:
- ssh-agent bash
> ```bash
ssh-agent bash
ssh-add id_rsa_work
```
## 修改config文件
> 在.ssh目录下找到config文件,如果没有就创建:
>
```bash
$ touch config   #创建config文件
```
```bash
#该文件用于配置私钥对应的服务器
#Default github user(kkaxiao@gmail.com)
Host github.com
HostName github.com
User git
IdentityFile c:/Users/admin/.ssh/id_rsa
#second User guanshuige@gmail.com #建立一个GITHUB的别名,新的帐号用这个别名进行clone 和 update
Host github2
HostName github.com
User git
IdentityFile c:/User/admin/.ssh/id_rsa_guanshuige
```
## 测试
```bash
$ ssh -T github2
$ ssh -T git@github.com
```
***
# 使用
- 将新的RSA公钥加入到GITHUB帐号后,即可用别名更新提交
```bash
$ git clone github2:guanshuige/guanshuige.github.io
$ git push
$ git pull
```






