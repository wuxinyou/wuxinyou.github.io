---
layout: post
title:  "Eos开发实例--众筹项目"
date:   2019-08-26 9:39:04
categories: blockChain
tags: EOS 原创
author: wuxy
---

* content
{:toc}

## 说明
- 本实例为一个众筹Dapp,界面简单，代码量很少，但是基本的Dapp开发的方法都概括了，非常便于入门者学习；
- 本文实例的开发环境和工具为：window10,nodejs,js4eos,react,eosjs,kylin测试网；
- 本实例的开发不需要安装EOSIO和EOSCDT;
- 本实例开发理论上可以用任何IDE，但是本文推荐使用vscode.
- 本实例是一个网页版的Dapp,这是Dapp主流开发方式，当然也可采用客户端的方式，这两种方式都可以，但是网页版的更加精简，方便教学和入门。
- 实例为什么不用主网测试呢？因为在主网上测试需要真金白银，而测试网上是免费的，而开发过程是一样一样的。

## 准备工作和前提条件
- 想要开发EOS Dapp必须精读白皮书和官网教程，看完之后可能还是云里雾里，没关系，我们可以有些概念，等用到的时候，就会恍然大悟，原来如此了。
- 详细了解js4eos和eosjs，简单了解react,这些东西是干什么的？先不用管他，后面的教程里面会提到;
- 了解一下kylin测试网，并且要在上面注册大于2个账户，注册办法很简单，官网有说明的；
- 基本了解前端开发流程，了解html，基本了解C++语法;
- 如果你完成了以上的准备工作，那么恭喜你，你只需要再用两天的时间，就可以实现一个简单的Dapp了。

## Dapp产品效果
这里先展示一下DApp的效果，大家有了目标效果，就有了方向，也对我们接下来要做的工作有个心理准备，知道我们要做什么，可以做什么。
- Dapp网页效果
查看Dapp网页：https://wuxyblockchain.github.io/
![Dapp网页](/images/2019-08-26-blockChain-EOSDappDev-offerlove-1.png)
- kylin测试网效果
查看智能合约：https://kylin.eosx.io/account/wuxytestnet1
![kylin测试网](/images/2019-08-26-blockChain-EOSDappDev-offerlove-2.png)

## 开发准备工作和说明
- 开发流程说明
EOS dapp开发主要分为2大块：前端开发+智能合约开发，看起来和普通的app开发基本上差不多，实际表面上确实如此，很多工作Eosio已经帮我们做好了，作为初级dapp开发者，我们可以不深究其中原理，等我们会使用了eos再去深究eos是怎么实现分布式的，所以作为一个初学者，请你们忘记它是一个分布式Dapp,把他当成一个简单的app应用去开发，你就会发现原来一切都如此简单。
- 安装node
进入node官网，安装windows版本的即可，和普通软件安装方法一样，简单免述,最新的node都已经包含了npm工具；无论是前端开发还是智能合约开发，node都是必要的。后面的开发过程会看到。
- 安装js4eos
使用npm安装即可，npm install js4eos ; npm npm安装详解可以自行百度;
具体详情可以查看：https://github.com/itleaks/js4eos

js4eos是用于开发智能合约的，其目的是取代了eosio和eoscdt，这就是为什么不用安装eosio的原因了，但是eos官方还是建议使用eos，开发命令基本一致。
- 安装eosjs
eosjs是EOS RPC库，有了eosjs后，js就可以和EOS通信了，所以eosjs是用于前端开发的。
安装方法和详细说明：https://github.com/EOSIO/eosjs

## DApp开发
从这里开始真正的Dapp 开发，Dapp开发分为前端开发和智能合约开发，其实者这两者的开发没有说明关系，完全可以相互独立，只是前端开发会调用智能合约里的action，所以我们的开发流程是先开发智能合约，再开发前端，其中智能合约是完全可以独立使用的，不过没有前端的话，只能用命令行，很不方便。
### 智能合约开发
智能合约开发，就是使用eos和C++开发，这里为了教学方便使用了js4eos，具体的使用方法可以参考：https://github.com/itleaks/js4eos，官网都非常详细，我不做过多的说明。当然也可以查看：js4eos开发智能合约，是我整理的开发步骤。

智能合约开发还是很简单的，主要步骤就是:
- 编写合约代码
- 编译生成wasm文件
- 编译生成abi文件
- 部署wasm和abi文件
到这里就完成了，接下来我们还可以测试一下，合约是否正常工作了。测试方法就是用命令行push调用合约，调用和即可在： https://kylin.eosx.io/account/wuxytestnet1 中看到合约信息。

以下是众筹合约的源码：

```
//2019-08-26
//众筹合约
//wuxy

#include <eosiolib/eosio.hpp>
#include <eosiolib/asset.hpp>

using namespace eosio;

class loveoffering:public eosio::contract
{
    public:
    loveoffering(account_name self):eosio::contract(self),_projects(self,self),_donations(self,self)
    {

    }

    /// @abi action
    void version()
    {
        print("loveoffering version 0.1");
    }

    /// @abi action
    void addproject(std::string projectName,std::string projectIntro,uint64_t projectToken)
    {
        require_auth(_self);
        print("Add project: ",projectName);
        print("Project Introduction: ",projectIntro);

        for(auto &item:_projects)
        {
            if(item.projectName==projectName)
            {
                print("Same Project Name: ",projectName);
                eosio_assert(true,"Same Project Name");
            }
        }

        _projects.emplace(get_self(),[&](auto &p)
        {
            p.key=_projects.available_primary_key();
            p.projectId=_projects.available_primary_key();
            p.projectName=projectName;
            p.projectStatus=1;
            p.intro=projectIntro;
            p.quantityWanted=projectToken;
        });
    }

    /// @abi action
    void modifyproj(std::string projectName,std::string param,uint64_t value)
    {
        require_auth(_self);  //只能是合约用户修改
        for(auto &item:_projects)
        {
            if(item.projectName==projectName)
            {
                if(std::string("quantityWanted")==param) //修改预期数量
                {
                    _projects.modify(item,get_self(),[&](auto &p){
                    p.quantityReceived=value;
                    });
                }
                else if(std::string("quantityReceived")==param) //修改已经收到的数量
                {
                    _projects.modify(item,get_self(),[&](auto &p){
                    p.quantityReceived=value;
                    });
                }
            }
        }
    }

    /// @abi action
    void offerlove(account_name donator,account_name to,asset quantity ,std::string projectName)
    {
        require_auth(donator);
        print("Donate for ",projectName," by ",donator);



        // for(auto &item:_donations)
        // {
        //     if(item.projectName==projectName && item.account==donator)
        //     {
        //         print(donator," has already donated ",projectName);
        //         eosio_assert(true,"already donated");
        //         return;
        //     }
        // }


        uint64_t projectId=99999;
        for(auto &item:_projects)
        {
            if(item.projectName==projectName)
            {
                projectId=item.projectId;

                //inline action
                // struct transfer_args
                // {
                //     account_name from;
                //     account_name to;
                //     asset quantity;
                //     std::string memo;
                // };

                // action(
                //     permission_level{donator,N(active)},
                //     N(eosio.token),
                //     N(transfer),
                //     transfer_args{
                //     donator,
                //     _self,
                //     asset(10000,S(4,EOS)),
                //     " memo "}
                //     ).send();

                action(
                    permission_level{donator,N(active)},
                    N(eosio.token),
                    N(transfer),
                    std::make_tuple(donator,to,quantity,std::string("donate successful"))
                ).send();

                //以下方法不知道为什么不可以
                // action(
                //     permission_level{donator,N(active)},
                //     N(eosio.token),
                //     N(transfer),
                //     std::make_tuple(donator,_self,asset(10000,S(4,EOS)),std::string("memo"))
                // ).send();



                _projects.modify(item,get_self(),[&](auto &p){
                    p.quantityReceived=p.quantityReceived+quantity.amount/10000;
                    });

                _donations.emplace(get_self(),[&](auto &d){
                    d.key=_donations.available_primary_key();
                    d.projectId=projectId;
                    d.projectName=projectName;
                    d.account=donator;
                });
            }
        }

        eosio_assert(99999!=projectId,"No Such Project");
    }

    /// @abi action
    void transfer(account_name from,account_name to,asset quantity)
    {
        eosio_assert(is_account(from),"from account does not exist");
        eosio_assert(is_account(to),"to account does not exist");
        eosio_assert(quantity.is_valid(),"invalid quantity");
        eosio_assert(quantity.amount>0,"quantity must >0 ");
        require_auth(from);
        action(
            permission_level{from,N(active)},
            N(eosio.token),
            N(transfer),
            std::make_tuple(from,to,quantity,std::string("memo"))
        ).send();


    }




    private:
    /// @abi table
    struct project
    {
        uint64_t key;
        uint64_t projectId;
        account_name projectOwner;
        std::string projectName;
        uint8_t projectStatus =0;
        std::string intro;
        uint64_t quantityWanted;
        uint64_t quantityReceived =0;
        uint64_t primary_key() const {return key;}
        uint64_t by_projectId() const {return projectId;}

    };

    typedef eosio::multi_index<N(project),project,indexed_by<N(projectId),const_mem_fun<project,uint64_t,&project::by_projectId>>> projectstable;

    /// @abi table
    struct donation
    {
        uint64_t key;
        uint64_t projectId;
        std::string projectName;
        std::string thanksTokenAddress;
        account_name account;

        uint64_t primary_key() const {return key;}
        uint64_t by_projectId() const {return projectId;}

    };
    typedef eosio::multi_index<N(donation),donation,indexed_by<N(projectId),const_mem_fun<donation,uint64_t,&donation::by_projectId>>> donations;

    projectstable _projects;
    donations     _donations;
};

EOSIO_ABI(loveoffering,(version)(addproject)(offerlove)(transfer)(modifyproj))
```

代码比较简单，但要求有C++基础，在C++的基础上添加了一点eosio的相关库，其中相关语法可以查看eos官网。

值得注意的是action和结构体前面的注释说明是必须的，即 "/// @abi table"和"/// @abi action",分别用于说明是table和action,如果不添加该注释，那么在abi文件中就找不到table和action.

### 前端开发

本实例的前端UI采用react架构，react仅仅是一个UI架构，dapp开发可以不使用react，但是node和eosjs是必须的。react的基本使用可以参考：[visual studio code + react 开发环境搭建](https://www.jianshu.com/p/ec7c2bab16cc)。

eosjs的基本使用可以参考：https://eosio.github.io/eosjs/，本文所使用过的到的eosjs接口都很基础，前面的链接里都可以看到详细说明，本文就不作赘述了。

下面是前端开发的核心代码,即app.js文件：

```
import React from 'react';
import logo from './logo.svg';
import './App.css';
import Background1 from './backg1.jpg';
import Background2 from './backg3.jpg';

import { Api, JsonRpc, RpcError } from 'eosjs';
import { JsSignatureProvider } from 'eosjs/dist/eosjs-jssig';

//5JcYhRpuN8KYg4sfxVe7ZUCYhQ3mn1tXXTYxJdopNi1oCGSGtLK  :wuxytestnet1
//5JdSAVq17bFaunKieb8Jod7WLP7u3NvSt2GMkPJJE1DFuKEXrbi  :wuxytestnet2
const defaultPrivateKey = "5JdSAVq17bFaunKieb8Jod7WLP7u3NvSt2GMkPJJE1DFuKEXrbi"; //
const signatureProvider = new JsSignatureProvider([defaultPrivateKey]);

//https://api.kylin.alohaeos.com
const rpc = new JsonRpc('https://api.kylin.alohaeos.com', { fetch });
const api = new Api({ rpc, signatureProvider, textDecoder: new TextDecoder(), textEncoder: new TextEncoder() });



//定义背景样式
var sectionStyle = {
  width: "100%",
  height: "100px",
  textAlign:'center',

};
//定义背景样式
var sectionStyle1 = {
  margin:0,
  width: "100%",
  height: "100px",
  textAlign:'center',
  backgroundImage: `url(${Background1})`
};
//定义背景样式
var sectionStyle2 = {  
  margin:0,
  padding:0,
  width: "100%",
  height: "800px",
  textAlign:'center',
  backgroundImage: `url(${Background2})`

};


class App extends React.Component {  

  constructor(props){
    super(props)
    //保存页面数据
    this.state = {
      info: {
        "server_version":"",
        "chain_id":"",
        "head_block_num":"",
        "last_irreversible_block_num":"",
        "last_irreversible_block_id":"",
        "head_block_id":"",
        "head_block_time":"",
        "head_block_producer":"",
        "virtual_block_cpu_limit":"",
        "virtual_block_net_limit":"",
        "block_cpu_limit":"",
        "block_net_limit":"",
        "server_version_string":""},

      projects:[{
        "key":"",
        "projectId":"",
        "projectOwner":"",
        "projectName":"",
        "projectStatus":"",
        "intro":"",
        "quantityWanted":"",
        "quantityReceived":""
      }]
    }
  }

  //Component的生命周期函数之一，在组件挂载到DOM节点完成后调用
  componentDidMount()
  {
    console.log('component');

    //get_info
    (async () => {
       rpc.get_info()
      .then(res => {
        //异步获取数据后，修改state，会调用render函数重新渲染页面
        //我们不需要进行DOM操作
        console.log("get");

        this.setState({info:res});        
      })
      .catch(error => {
        console.log("error at get ")
        console.log(error);
      });
    })();

    //get_table_rows
    (async () => {
      rpc.get_table_rows({
        json: true,              // Get the response as json
        code: 'wuxytestnet1',     // Contract that we target
        scope: 'wuxytestnet1',         // Account that owns the data
        table: 'project',        // Table name
        limit: 10,               // Maximum number of rows that we want to get
        })
        .then(res => {
          let rows=res.rows;
          this.setState({projects: rows});  
        });
    })();



  }

  getProjectInfo(){
    //get_table_rows
    (async () => {
      rpc.get_table_rows({
        json: true,              // Get the response as json
        code: 'wuxytestnet1',     // Contract that we target
        scope: 'wuxytestnet1',         // Account that owns the data
        table: 'project',        // Table name
        limit: 10,               // Maximum number of rows that we want to get
        })
        .then(res => {
          let rows=res.rows;
          this.setState({projects: rows});  
        });
    })();
  }

  //捐献函数
  donate(){
    console.log("donate...")
    let projecttoDonate=document.getElementById("projecttoDonate").value;  //视频里面是.value，但是这里没有value的选项；
    let contractSender=document.getElementById("contractSender").value;
    //let privateKey = document.getElementById("privateKey").value;
    (async () => {
      const result = await api.transact({
        actions: [{
          account: 'wuxytestnet1',
          name: 'offerlove',
          authorization: [{
            actor: contractSender,
            permission: 'active',
          }],
          data: {
            donator:contractSender,
            to:'wuxytestnet1',
            quantity:'10.0000 EOS',
            projectName:projecttoDonate

          },
        }]
      }, {
        broadcast:true,
        sign:true,
        blocksBehind: 3,
        expireSeconds: 30,
      });  

      console.dir(result);
      this.getProjectInfo();

    })();


  }

  version(){
    (async () => {
      const result = await api.transact({
        actions: [{
          account: 'wuxytestnet1',
          name: 'version',
          authorization: [{
            actor: 'wuxytestnet2',
            permission: 'active',
          }],
          data: {

          },
        }]
      }, {
        blocksBehind: 3,
        expireSeconds: 30,
      });   

      console.log("check version");
    })();
  }



  //渲染函数，每一个组件都必须实现，Component生命周期函数之一
  render() {

    return (
      //render函数不能返回多个元素，用div包起来
      <div style={sectionStyle}>
        <div style={sectionStyle1}></div>

        <div style={sectionStyle2}>

          {  
            //遍历并显示projects        
            this.state.projects.map(function(project,index){
                return <div key={index}>
                    <h2>Project: {project.projectName}</h2>
                    <p>Prokect Intro: {project.intro}</p>
                    <p>Prokect Id: {project.projectId}</p>
                    <p>Prokect Status: {project.projectStatus}</p>
                    <p>Prokect Wanted: {project.quantityWanted}</p>
                    <p>Prokect Received: {project.quantityReceived}</p>
                    </div>
            })
          }


          <form id="donate-form">
              <p>请输入您要捐献的项目名称</p>
              <input type="text" name="projecttoDonate" id="projecttoDonate"></input>
              <p>请输入您的Eos账户名：</p>
              <input type ="text" name="contractSender" id="contractSender"></input>
              <p></p>
              <button type="button" onClick={this.donate.bind(this)}>捐献10EOS</button>


          </form>
        </div>


        <div style={sectionStyle1}></div>





        {/* //显示kylin测试网的Info
        <form>
          <h1>kylin Blockchain Info：</h1>

          <h3><span>server_version：</span>{this.state.info.server_version}</h3>
          <h3><span>chain_id：</span>{this.state.info.chain_id}</h3>
          <h3><span>head_block_num：</span>{this.state.info.head_block_num}</h3>
          <h3><span>last_irreversible_block_num：</span>{this.state.info.last_irreversible_block_num}</h3>
          <h3><span>last_irreversible_block_id：</span>{this.state.info.last_irreversible_block_id}</h3>
          <h3><span>head_block_id：</span>{this.state.info.head_block_id}</h3>
          <h3><span>head_block_time：</span>{this.state.info.head_block_time}</h3>
          <h3><span>head_block_producer：</span>{this.state.info.head_block_producer}</h3>
          <h3><span>virtual_block_cpu_limit：</span>{this.state.info.virtual_block_cpu_limit}</h3>
          <h3><span>virtual_block_net_limit：</span>{this.state.info.virtual_block_net_limit}</h3>
          <h3><span>block_cpu_limit：</span>{this.state.info.block_cpu_limit}</h3>
          <h3><span>block_net_limit：</span>{this.state.info.block_net_limit}</h3>
          <h3><span>server_version_string：</span>{this.state.info.server_version_string}</h3>
        </form> */}
      </div>
    );
  }



}  //end of class

export default App;

```

完成了前端代码的编写，还需要npm run 编译以下js文件，然后就可以发布到你的服务器上了，本实例使用的是github免费的服务器。

## 总结
本实例完整的开发了一个简单的Eos dapp，从智能合约开发到前端开发，可以看到，其实dapp开发并没有我们想想的那么神秘，甚至比传统的dapp开发还要简单。当然本文在讲述过程中不自主的跳过了很多细节，但是我都给出了说明和引导，因为我觉得这些细节必须由读者亲自去挖掘才由意义。如有疑问可以联系作者，作者致力于开源社区。

## 参考资料
以下是本实例开发过程常常用到的资料和工具，建议开发者都了解一下以下资料。
- http://blog.eosdata.io/ :EOS区块链开发指南
- https://www.cryptokylin.io/ ：kylin测试网官网
- https://www.jianshu.com/p/ec7c2bab16cc ：visual studio code + react 开发环境搭建
- 《EOS区块链应用开发指南》虞家男
- 深入理解EOS原理解析与开发实战
