---
layout: post
title:  "js4eos开发智能合约"
date:   2019-08-01 16:54:54
categories: blockChain
tags: EOS 原创
author: wuxy
---

* content
{:toc}

## 说明
- 本文测试环境为：window10,js4eos,kylin测试网
- 所有命令都是在powershell中运行的，在命令中遇到“”时需要加“\ ”转义；
- 当然也可以在mingw等linux的环境下运行，比如git bash就可以，就不需要转义了。
- 预先安装node,安装过程很简答，直接到node官网，然后和普通软件安装方式一样；
- 然后安装js4eos.
- js4eos的命令和cleos基本一致，具体参见js4eos官网。
- 预先创建kyiln账号，免费的。


## 开发步骤

### 钱包账号
- js4eos wallet create -n wuxyWallet
- js4eos wallet unlock -n wuxyWallet
- js4eos wallet import -n wuxyWallet 5JcYhRpuN8KYg4sfxVe7ZUCYhQ3mn1tXXTYxJdopNi1oCGSGtLK
- js4eos wallet keys：查看钱包中的公钥
- js4eos get account wuxytestnet1:查看账户信息

### 编写编译智能合约
- mkdir loveoffering :一定要是英文目录
- cd loveoffering
- 新建智能合约源文件：loveoffering.cpp,其中.hpp文件可选，简单的合约我们可以把所有的代码写在.cpp里
- js4eos compile -o loveoffering.wasm loveoffering.cpp:编译生成wasm文件，在区块链中实际运行的文件，机器看的。
- js4eos compile -g loveoffering.abi loveoffering.cpp:编译生成abi文件，作为合约接口，给人看的，方便调用。
- 合约名就是账户名，实际上不存在合约名这个概念，账户名、文件名、类名3个名称没有关系。
- action和table的命令要符合base32的规则，即a~z,1~5

### 部署合约
- js4eos config set --network kylin:把当前的网络设置成kylin
- js4eos config set：可查看网络
- 切换到loveoffering目录上一级
- js4eos set contract wuxytestnet1 loveoffering:其中loveoffering是文件夹
- 部署需要私钥，所有记得wallet unlock

### 调用合约
- js4eos push action wuxytestnet1 version '[]' -p wuxytestnet1@active:因为version action没有参数
- js4eos push action wuxytestnet1 addprooject '[\"project1\",\"help_child\",100]' -p wuxytestnet1@active:在ps中需要添加转义
- js4eos push action wuxytestnet1 offerlove '[\"wuxytestnet2\",\"knights\"]' -p wuxytestnet2@active
- js4eos push action wuxytestnet1 modifyproj '["knights","quantityReceived",0]' -p wuxytestnet1@active


## 其他配置
 配置合约账号的eosio.code权限：

- js4eos set account permission wuxytestnet1 active '{\"threshold\": 1, \"keys\": [{\"key\":\"EOS5SCyJoqf1t114JAFPRW7AMbY6q2F1rVeTyFFbpfkuHg6nZwi7r\",\"weight\": 1}], \"accounts\": [{\"permission\":{\"actor\":\"wuxytestnet1\", \"permission\":\"eosio.code\"}, \"weight\":1}], \"waits\": []}' owner -p wuxytestnet1@owner

- js4eos set account permission wuxytestnet2 active '{\"threshold\": 1, \"keys\": [{\"key\":\"EOS7v3ezbBTbLSJr8cRkeySGq8c1x7YouyVQin6k82e6unbtTvN3S\",\"weight\": 1}], \"accounts\": [{\"permission\":{\"actor\":\"wuxytestnet1\", \"permission\":\"eosio.code\"}, \"weight\":1}], \"waits\": []}' owner -p wuxytestnet2@owner

- js4eos push action eosio.token transfer '[ \"wuxytestnet2\", \"wuxytestnet1\", \"25.0000 EOS\", \"memo\"]' -p wuxytestnet2@active：转账操作
## 参考文献
- https://github.com/itleaks/js4eos ： 官方资料
- 一条Javascript命令玩转EOS, js4eos开源了：https://blog.csdn.net/ITleaks/article/details/82466329：其实就是翻译官网资料，翻译的还不错
