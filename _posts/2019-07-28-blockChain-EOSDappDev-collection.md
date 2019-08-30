---
layout: post
title:  "EOSDapp 开发-收录"
date:   2019-07-28 9:04:54
categories: blockChain
tags: EOS 收录
author: wuxy
---

* content
{:toc}

## EOSDapp开发综合博客
该类博客包含EOS开发的前端+后端开发。

 - [EOS开发系列目录（2019版）](https://shimo.im/docs/jt2MhxPYTKI6tWnp/read)：网友的个人博客
 - [](https://shaokun11.github.io/)：steven网友的个人博客
 - https://www.chaindesk.cn/witbook/37/649：EOS DApp合约开发之DICE游戏(线上完整项目)，非常详细的开发，包含源码，问题和解决方案。
 - https://learnblockchain.cn/2019/05/14/eosdice-random1/：EOS DApp 随机数漏洞分析 1 - EOSDice 随机数被预测：深入浅出区块链，系统学习区块链技术


## 视频教程
 - https://ke.qq.com/course/345101?term_id=100410155&taid=2929541358371853：eos众筹Dapp开发实例教程，共10集，比较详细。教程基于js4eos开发智能合约，基于eosjs开发前端。本人已看完了所有视频教程。

## 前端开发
- [visual studio code + react 开发环境搭建](https://www.jianshu.com/p/ec7c2bab16cc)
- https://code.visualstudio.com/docs/nodejs/reactjs-tutorial ：visual studio code + react 开发环境搭建
- https://developers.eos.io/eosio-nodeos/reference#get_account-1 ：RPC接口说明
- http://cw.hubwiz.com/card/c/eos-rpc-api/: EOS RPC API手册 - 汇智网

## 智能合约开发
- EOS源码分析]8.EOS保留权限eosio.code深度解读：https://blog.csdn.net/ITleaks/article/details/80557560：解决了eosio.code 的授权问题。
- eosio.code权限使合约间交互：https://www.chaindesk.cn/witbook/37/733：用实例解读了eosio.code权限问题，但是在添加授权的命令忘记了添加 owner -p wuxytestnet1@owner.但是其online action有一定的参看价值。其次该网站具有很全的EOS开发教程。
- EOS智能合约开发及授权（一）转账：https://www.jianshu.com/p/868a3f59b8e9，亲测可行。




## 开发工具

### eosjs
- https://eosio.github.io/eosjs/: eosjs使用教程
- https://github.com/EOSIO/eosjs：eosjs官方库

### js4eos
- https://github.com/itleaks/js4eos :

### React
- https://react.docschina.org/: React官方网站

## 区块链网络

### 主网
- 获取EOS主网信息：https://api.eosnewyork.io/v1/chain/get_info

### kylin测试网
- 获取kylin信息:http://kylin.meet.one:8888/v1/chain/get_info
- "httpEndpoint": "http://kylin.meet.one:8888",
  "httpEndpoints":
  "http://39.108.231.157:30065",
  "http://api.kylin.eoseco.com",
  "https://api-kylin.eoslaomao.com"
