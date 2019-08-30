---
layout: post
title:  "一台电脑上同时是使用两个github账户"
date:   2019-07-27 15:04:54
categories: git
tags: git 编整
author: wuxy
---

* content
{:toc}

## 背景
我2个github账号，有一次我想分别拉取和上传这2个github账号上的仓库，我发现只能操作一个，后来发现是需要设置的。

## 基本原理

- 一个github账号需要对应一个ssh-key,同样的一个ssh-key只能为一个github账户所用；
- 所以我们要生成2个ssh-key，然后将这两个ssh-key分别添加至github账户上；
- 可以通过git config user.name却换github账户；

## 解决办法
### 生成2个ssh-key

为了举例方便，这里使用“one”和“two”两个账户。下同。

$ ssh-keygen -t rsa -C "one@gmail.com"

$ ssh-keygen -t rsa -C "two@gmail.com"

不要一路回车，分别在第一个对话的时候输入重命名（id_rsa_one和id_rsa_two），这样会生成两份包含私钥和公钥的4个文件。

注1：ssh-keygen是linux命令，可以让两个机器之间使用ssh而不需要用户名和密码

注2：一定要在~/.ssh路径下运行命令行，不然生成的文件不会出现在当前目录

注3：在win10下，一般的路径为：C:\Users\DELL\.ssh


### 添加私钥

1、打开ssh-agent


(1)如果你是github官方的bash：

$ ssh-agent -s

(2) 如果你是其它(实测在win10上就是用此命令)，比如msysgit：

$ eval $(ssh-agent -s)



2、添加私钥

$ ssh-add ~/.ssh/id_rsa_one

$ ssh-add ~/.ssh/id_rsa_two

### 创建config文件
在在当前目录(.ssh路径下)想创建config文件，vi config;

    one(one@gmail.com)

    Host one.github.com

　　HostName github.com

　　User git

　　IdentityFile ~/.ssh/id_rsa_one

    two(two@ gmail.com)

    Host two.github.com

　　HostName github.com

　　User git

　　IdentityFile ~/.ssh/id_rsa_two



### 部署SSH key
分别登陆两个github账号，进入Personal settings –> SSH and GPG keys：

点击"new SSH key"， 把下面两个公钥的内容分别添加到相应的github账号中。

### 远程测试

$ ssh –T one.github.com

$ ssh –T two.github.com

一般你会看到如下，注意大小写敏感：

$ ssh -T wuxyBlockChain
Hi wuxyBlockChain! You've successfully authenticated, but GitHub does not provide shell access.

### 使用

1、clone到本地



(1)原来的写法：

$ git clone git@github.com: one的用户名/learngit.git

(2)现在的写法：

$ git clone git@one.github.com: one的用户名/learngit.git

$ git clone git@two.github.com: two的用户名/learngit.git



2、记得给这个仓库设置局部的用户名和邮箱：

$ git config user.name "one_name" ; git config user.email "one_email"

$ git config user.name "two_name" ; git config user.email "two_email"



3、上述都成功后，会发现钥匙会由灰变绿。
![产品路线图](/images/2019-07-27-git-useMultiGitAtOnePC-1.png)

### 常见问题
拼写错误：

$ git clone git@wuxyBlockChain:wuxyBlockChian/wuxyBlockChain.github.io.git
Cloning into 'wuxyBlockChain.github.io'...
ERROR: Repository not found.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

$ git clone git@wuxyblockchain:wuxyBlockChian/wuxyBlockChian.github.io.git
Cloning into 'wuxyBlockChian.github.io'...
ssh: Could not resolve hostname wuxyblockchain: Name or service not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.


ping github失败

Ping不通，这时候，只需要在host文件里做些修改就可以，首先，定位到路径

C:\Windows\System32\drivers\etc

找到hosts文件，右键-属性-安全-编辑，选中当前电脑登录的用户，给自己最高权限，确认。

192.30.253.113    github.com

192.30.252.131 github.com

185.31.16.185 github.global.ssl.fastly.net

74.125.237.1 dl-ssl.google.com

173.194.127.200 groups.google.com

192.30.252.131 github.com

185.31.16.185 github.global.ssl.fastly.net

74.125.128.95 ajax.googleapis.com

保存，再ping，发现速度杠杠的.



## 参考文献
- https://www.cnblogs.com/xjnotxj/p/5845574.html：一台电脑上的git同时使用两个github账户
- https://www.jianshu.com/p/f2bef9737a8a：同一台设备如何使用两个GitHub帐号？
- https://blog.csdn.net/u012552275/article/details/61654857: 解决无法Ping通Github
