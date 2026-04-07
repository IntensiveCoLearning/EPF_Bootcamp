---
timezone: UTC+8
---

# ccboomer

**GitHub ID:** ccboom

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
今天进入EL层的具体学习

这可以比喻成一个账本记账的过程，只是比喻

在讲图之前，我们先看两个词：

1.状态：可以理解为总账，里面记录了此刻所有人的账户余额，合约的代码等等

2.区块：新一页的账单，这页纸上写满了大家刚发起的转账记录

所谓的状态转换，就是旧的状态加入新的区块，形成新的状态

我们先看看这个图  

![](https://epf.wiki/images/el-specs/stf_eels.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775567860744-image.png)

我们分析一下图：

1.准备材料：在图的左边，你得有俩新东西

新账单：里面包含了区块头，交易记录等待

旧账本：这是处理新帐单之前的账本状态，里面包含了当前区块链的历史以及当前的状态

2.开始处理：

也就是中间的大全，它是一个程序

验证区块是否合理，看看新的交易有没有问题，签名对不对

如果不对，那么就走下方，这页账单直接作废，账本状态不变

如果没问题，那就走右边的箭头，正式把这个账单放入账本中

这时候还有一个操作 Discard blocks preceding the recent 255：为了系统的效率，内存里面只保留最近的255区块的详细状态，太老的话就清理掉

3.产出结果，也就是图的右边

经过程序处理之后，我们就得到了图最右边的BlockChain' 和 State'

右上角多了一个 ' ，在数学和计算机上，代表新版本，也就是产生了新账本和新状态  
  
  
再看看这个下面的公式吧，其实就是上面图的过程  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775567882730-image.png)

σt：旧的状态，处理之前的旧帐本，图上左下角那个

B：当前的区块，就是刚刚那个新的账单

Π：状态转换函数：也就是那个严格的程序，负责核对和计算

σt+1：新的状态，处理完之后的最新账本  
​解读一下就是：程序用旧帐本和新帐单操作完后，算出一个新的账本

这样就容易理解一点

接下来学习一下底层数学逻辑和代码的实现

这里有一个难点是：折叠

什么是折叠？就是把全球所有的账户的状态，使用Hash计算压缩为一个短短的字符，即root hash，并保证只要有值改版，这个就会改变

![](https://epf.wiki/images/el-specs/state.png)

先来学习数学原理

黄皮书上的状态 σ 和Python代码里的State不是一回事。

数学上的状态不是某个具体固定的值，而是通过状态折叠函数动态算出来的

也就是：把所有的数据打包，生成的一个防伪码

一步一步来：

1.

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568607222-image.png)

这是处理单个账户内部的数据，把这个账户下所有的值，通过Trie结构压缩，算出一个指纹

算完之后用户状态变成了：nonce/balance/storageRoot/codeHash 四个状态

这就是中间的Account State

2.

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568616405-image.png)

把所有不空的账户收集起来，准备进行最终打包

3.

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568623288-image.png)

生成旧帐本的总指纹，把刚刚收集的所有信息塞到Trie里向上计算，最终得到一个根节点

4.

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568630590-image.png)

拿着新账单算完之后的状态，把这个状态再走一遍折叠流程，算出当前区块全新的根  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568682636-image.png)

然后我们看看代码层面的步骤

当系统收到一个新的区块，必须按照以下步骤来检查和执行：

1.拿旧账本：找出当前链上最新的区块头

2.查手续费：检查Blob数据的相关Gas是否计算正确

3.查新账单格式：看看格式，时间和父区块头对比是不是合法的

4.查废弃字段：检查Ommers字段是不是空的，转为Pos之后，这个字段就必须为空了

5.开始计算：这是最重要的一步，系统去执行新区块里的所有交易，执行完产出几个东西

即：Gas used，Trie Root，Logs Bloom，State

6.核对结果：刚刚计算出来的指纹，和别人打包过来里写的State\_root对比，对不上说明造假了

7.正式入库：所有审核都通过，把这个块正式链接到链末端

8.清理内容：超过255的旧状态从内存中删除

9.错误处理：如果上面任何一个环境出现问题，立刻报错，抛出 "Invalid Block"，拒绝这个新块

这张图详细展示了以太坊如何把海量数据提炼成一个Trie Root，并且制定9步审查，确保每一个交易每一个新块都准备无误的添加到总账本之中
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->

今天学习了Ethereum的发展历史，从网络的发展到后面的密码学，然后发展出来bitcoin 后面到Ethereum，一个很线性的介绍，告诉我们到底从开始到以太坊的发展历程

还学习了协议的架构，有两层

-   一层是 EL也就是 execution layer，这一层主要是处理用户的交易，是执行层
    
-   另一层是 CL也就是 consensus layer,这一层主要是维持共识，确保节点的机制是一致的，保持安全
    

接下来了解了一下Ethereum的设计理念，分别是：

-   简洁性：当时想的是设计一个简单的规则，让一个及格水平的程序员也能看懂以太坊的全部代码。但是随着时间的发展，这一部分好像并没有想象中那么好我感觉
    
-   通用性：以太坊不是想做几个固定的功能，而是做一个平台，给开发者使用，然后开发者可以通过自己的逻辑来构造东西。这和BTC有所不同，BTC更像一个账本，但是以太坊像是一个可以开发工具的平台
    
-   模块化：这个很好理解，拆开分为不同模块，可以实现部分的转换而不影响全局
    
-   非歧视性：其实就是中立性，不会主关评判到底是好的用户还是坏的用户，什么东西乱七八糟的，哪怕你写一个无限循环也没问题，只要一直支付gas
    
-   敏捷性：就是可升级，如果发现更好的技术，可以通过EIP进行升级
    
    总结一下就是说做一个人人都能看懂的，啥也能干的，不审查并且能升级的世界级计算机
    

下面重点看一下协议的几个重要转变阶段：

1.  Frontier 是主网上线前的第一个版本。是在创世区块的时候产生的，当时以太坊还是非常新的系统，所以要让大家参与进来，然后大家开发开发之后，发现新的问题，它更像早期给开发者的试用版。当时设置gas limit只有**5000**，这放到现在什么都干不了。还有一个叫**Canary contracts** 的合约信号，当网络出现重大升级，就会把值改为1，这时候矿工就会下载新版本运行，避免了长时间不对而导致的链割裂。
    
2.  **Homestead** 是第二个重大版本，从前期的测试逐步趋于稳定。主要引入了三个EIP
    
    -   EIP-2：提高通过普通交易创建合约的Gas，早期合约部署太便宜了，定价不合理，提高之后会趋于平衡。防止签名篡改，修复了一个密码学的bug，ECDSA 签名里有一些参数，其中一个叫s，在不改变交易的情况下能改变hash值，现在强制大家使用一种规则，修复了这个bug。以前如果gas不够，还会留下一个空壳合约，现在创建失败就直接失败了。而且还调整了出块的算法。
        
    -   EIP-7：新增了一个操作码:DELEGATECALL,这个非常重要，可以使用代理合约，我的github里有详细解释，这里不再赘述
        
    -   EIP-8：以太坊要升级，每次升级之后老客户端看不懂新的数据，网络就会变得非常脆弱，这个是规定对于数据放宽一些，不要动不动就报错，忽略就行了
        
3.  **The Merge** 这个指的是从POW转型POS  
    为什么叫合并？因为不是单纯的从一条链转到另一条，相当于使用了原来的执行系统，加上新的共识POS系统，共识层和执行层分离更清晰，转向权益证明。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
