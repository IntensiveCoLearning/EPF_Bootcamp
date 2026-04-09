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
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
接着昨天的继续往下学习，太困难了

接下来我们看 Exchange of Capabilities

engine\_exchangeCapabilities 不是同步链数据，它是在协商接口能力

从正常开始运行之前，CL和EL会通过engine\_exchangeCapabilities做一次 capability exchange

这一步是协商两步支持哪些Engine API方法，目的是为了让双方在共同的协议版本上工作，保证兼容性并保留向后兼容的前提下使用新特性

先确定两方会用哪一个接口语言，然后再展开合作

CL和EL不是默认匹配的，我们要把两边对齐，不然之后调用方法可能会出现问题

这一步有三个意义：

保证CL和EL能通信，支持协议升级带来的新接口版本，不至于升级之后直接淘汰旧的。

Node Startup是什么

其实就是讲述一个节点刚启动的时候CL和EL怎么进入可工作状态的

CL调用engine\_exchangeCapabilities，EL给它返回支持的方法，然后CL发一次不带 payload attributes的engine\_forkchoiceUpdated，然后EL根据同步的进度反馈状态

重点不是让EL新造区块，而是先把EL的链视图对齐到当前的fork choice

节点启动不是EL自己单独启动，而是EL和CL协同完成初始化

Validator Operation在讲什么

实际上讲了验证者日常运行的两条工作线，太复杂了就不说了，我也说不清

其实就算在自己出块的时候，EL负责构建，收到别人区块的时候，EL负责验证

接下来进入 Components of the Architecture

先看看Engine

原文说 EL client acts as an execution engine

它暴露一个经过认证的Engine API端口给 CL，这个engine也被叫做 external consensus engine

一个EL只能被一个CL驱动，但一个CL可以链接多个EL

ENgine API走HTTP上的json-rpc，认证方式是 JWT，这个 Engine JSON-RPC 不对CL之外的地方开放

EL不只是被动接受数据，他对CL来说就是一个 execution engine

CL决定链怎么走，但是EL负责把 execution的事情做多，所以ENgine是CL的执行后端相当于

再看看 Routines -> Payload validation

这块说的是 Merge后，EL在协议中大概还剩什么工作，核心就是 state transition function

CL负责共识推进，EL负责把executin payload真正验证并执行成状态转换

太复杂了，解释不清

merge之后，EL的角色大幅度简化了，EL就是执行 state transition function

CL呢？后面学到再说吧

看代码：

func stf(parent types.Block, block types.Block, state state.StateDB) (state.StateDB, error) {

if err := core.VerifyHeaders(parent, block); err != nil {

return nil, err

}

for \_, tx := range block.Transactions() {

res, err := [vm.Run](http://vm.Run)(block.header(), tx, state)

if err != nil {

return nil, err

}

state = res

}

return state, nil

}

一个go简化的伪代码，基本就是EL的主要功能

先验证header，gas limit是不能乱搞，只能在父区块的 gas limit 的 1/1024 范围内调，block number必须按照顺序递增，EIP-1559的base fee必须按照规则更新

在EL里，区块不是交易了就能跑了，header约束本身就算有效性的一部分

再之后就可以执行交易了，执行的时候会把block header一起传进EVM，因为交易执行要上下文比如timestamp什么的，然后逐一执行，任何一笔失败，全部的交易都不能执行，块被判定喂invalid。只要有一个错误，就会污染整个区块。

然后每执行完一笔，state都会更新，所以后面的交易看到的不是父状态，而是前面交易积累过的状态

接下来讲的是Geth 分为4块

并不是深入讲解geth，只是把前面的概念在geth身上说一下

Transaction Execution in Geth：Geth会验证签名/nonce/gas费用，然后更新state，交易先进入mempool，再拿来打包

Block Processing & State Updates：多笔交易按照顺序执行，执行完后提交最终状态，并把state root写出来

Networking & Peer-to-Peer Communication：Geth用DevP2P交换交易和区块，新交易传播的时候，每个验收者先验证再转发

EVM Execution：Geth的核心执行单元依然是EVM，合约交互交易都在EVM内执行

接下来看 State Sync

EL的同步核心不是只追区块的高度，而是那倒足够可信/足够完整的当前 world state

这就是为什么一直强调state，因为对EL来说，要验证新区块，要构建新区块，那就必须知道当前的 world state是什么。不是只靠区块头能的出来的。

所有 execution clients 都需要最新的 world state 来验证和构建区块

新节点启动会用DevP2P 子协议

eth/\* 拿headers/bodies/recipts

snap/1 做state snapshots

因此客户端有俩选择

full sync 和 snap sync

Full Sync：也就是每一个状态都在你本地自己算出来，从genesis开始，逐渐重建本地的Trie，代价也非常大，时间长，消耗高，。这是信任最少的方式但是也是最慢的方式，EIP-4444完整实现后，不再支持从genesis full sync了

Snap Sync：把某个新的状态快照抓下来，然后补齐没有的。

所以 full sync 和 snap sync 的差别，不只是“快一点慢一点”，而是方法论不同：

full sync：从历史推到现在

snap sync：先拿到一个接近现在的状态快照，再修补到可用

snap sync 6步：

1.选一个最近 finalized 的 pivot block

2.通过 GetBlockHeaders 拿它的 header，读出 stateRoot

3.再拿到到 pivot 为止的 headers，明确目标 state root

4.分块下载 state trie / storage trie 的 leaves

GetAccountRange 拿 account leaves

GetStorageRanges 拿 storage slot leaves

5.用 GetByteCodes 下载各账户对应的 contract code

6.把这些 leaves 写进新的 snapshot DB，并用 Merkle range proofs 对照 pivot 的 stateRoot 验证

在这六步之后不是结束了，而是刚刚开始

难点变成了 快照完整性检验 缺失数据补齐 把snapshot format 还原喂可验证的trie 追赶最新区块

这就是 Healing Phase，它不是单纯的快，只是把慢的地方换了一个位置

下面看看 Transaction Pools吧

主要有两种 tx pool

Legacy pool 更像传统 gas-price / tip 竞争场景

Blob pool 是为 blob transaction 的特性单独设计的

太难了，太难了
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->

今天学习以太坊的架构

现在的以太坊EL，不再负责判断哪条链胜出，他负责把交易执行好，把状态算对，把数据存好，把网络连同通，把结果准确的交给CL

我们先说一下图上在说什么

其实就是一条控制链路

1.共识层CL在上面做共识决策

[2.Cl](http://2.Cl)通过Engine API来驱动执行层EL

3.EL内部负责交易，验证区块，维护状态，维护交易池

4.EL下面通过DevP2P和别的EL节点通信

5.DevP2P最初靠 bootnodes帮你进网，然后再发现更多peers

6.一旦CL通过 forkchoiceUpdated 告诉EL当前应该跟哪条链，EL就回去按需要取peer里拉区块，拉状态，做同步

7.钱包和Dapp不是走 Engine API，而是走 JSON-RPC API跟EL交互

这一段的关系是：

CL驱动EL，EL依赖DevP2P去接触网络，boot nodes只是入网入口，同步动作会和Engine APi 调用联动

共识层告诉执行层现在该看哪条路径，该不该开始构建payload，执行层再通过p2p网络把需要的区块和状态拿回来

也就是说，这张图里至少有三条关系同时存在：

控制关系：CL -> Engine API -> EL

网络关系：EL <-> DevP2P peers

启动入口：bootnodes -> 帮 EL 先连进网络

bootnodes 不是“中心服务器”，也不是“共识权威”，而只是一个初始的接入点

然后我们先看6个基础构件，有了这几个基础构建我们就能搭建最小的心智模型

1.EVM

虚拟化执行引擎，真实硬件有不同的CPU架构和指令集，同一程序在不同硬件上可能行为依赖底层实现，计算机会用虚拟机来抽象这些差异

EVM就是Ethereum程序的虚拟化执行引擎，他的作用是让不同硬件上得到一致结果，从而支持客户端之间形成共识。

实际就是强调两件事

1.EVM的核心价值是确定性，不同机器跑出的结果相同

2.EVM是抽象层，solidity不是底层执行格式，真正被执行的是编译后的EVM bytecode

如果没有EVM这种统一的虚拟执行层，那么不同客户端，不同操作系统，不同硬件算出来的交易可能是不同的结果，那么链就没办法共识

EVM不是solidity，也不是整个EL

2.State

状态机，Ethereum 是 general purpose computational system，它作为一个state machine运行，他会更具输入在不同状态之间转换

和Bitcoin不同，Ethereum 维护的是global state，包括地址，余额，合约代码，Merkle-Patricia Trie等

state是一个很大的合集，Ethereum不是简单的记住交易列表就行了，而是维护一份全局状态，并且不断的做状态变换，

3.Transsactions

触发状态转换的输入，就是交易。

交易不是目的，交易是输入，真正的核心动作是状态转换

有一个状态机，交易是喂给状态机的输入，EVM执行输入，如果输入合法，则得到新的状态

4.DevP2P

EL之间的通信接口，交易不是凭空全网同步，要靠EL节点之间相互传播，传播不是盲目传播，接受的时候会做是否合法的检测

钱包把交易给某个EL，EL放进mempool里面，它通过DevP2P发给卡EL peers，别的EL验证之后继续转发

所以DevP2P承担的就是执行层节点之间的数据传播

5.JSON-RPC API

钱包或Dapp与 EL 的通信，是通过标准化的 JSON-RPC API 进行的。

如果你问余额，区块，日志等，这都属于 JSON-RPC 侧，它面向的是 钱包，DApp，基础设施和外部程序

注意 JSON-RPC API != Engine API

6.Engine API

这个是EL与CL的内部接口，Engine API 不是对公众开放的通用 RPC。

CL与EL不能随便通信，他们是通过Engine API 协作，其中最关键的就是 告诉 EL 当前 fork choice / 是否开始 build payload，把 execution payload 交给 EL 验证

如果 JSON-RPC 是“用户调用 EL”，那 Engine API 就是 CL 调用 EL。

所以从这页 Overview 的排布上你已经能看出一个清晰分工：

钱包 / DApp -> JSON-RPC -> EL

CL -> Engine API -> EL

EL <-> DevP2P <-> 其他 EL  
  
漏了一个sync

EL不是自己随便同步的一条链，而是跟着CL选出来的fork choice去同步，验证，追赶

这里说的sync不是说网络下载问题，而是一个CL与EL协作的问题

CL先决定哪里当链的头，然后通过forkchoiceUpdated 告诉EL，EL再去peers那里拉缺的区块，拉回来后在EVM里面验证，最后吧自己当前能否处理这条链的状态反馈给CL

如果只看EL，它可能只知道有哪些块，有别的给我发送了什么数据，我现在状态库是什么样子，但是它不知道全网共识应该谁是head。

这个决定，PoS上主要是CL的工作

同步不是EL独立完成的，而是CL先给方向，EL再去同步和验证

同步包含两步：下载 remote blocks，在EVM中验证他们
<!-- DAILY_CHECKIN_2026-04-08_END -->

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

经过程序处理之后，我们就得到了图最右边的BlockChain’ 和 State’

右上角多了一个 ’ ，在数学和计算机上，代表新版本，也就是产生了新账本和新状态

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

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568607222-image.png)

这是处理单个账户内部的数据，把这个账户下所有的值，通过Trie结构压缩，算出一个指纹

算完之后用户状态变成了：nonce/balance/storageRoot/codeHash 四个状态

这就是中间的Account State

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568616405-image.png)

把所有不空的账户收集起来，准备进行最终打包

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-07-1775568623288-image.png)

生成旧帐本的总指纹，把刚刚收集的所有信息塞到Trie里向上计算，最终得到一个根节点

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

9.错误处理：如果上面任何一个环境出现问题，立刻报错，抛出 “Invalid Block”，拒绝这个新块

这张图详细展示了以太坊如何把海量数据提炼成一个Trie Root，并且制定9步审查，确保每一个交易每一个新块都准备无误的添加到总账本之中  
  
这一部分数学方面对于我来说实在太困难了，只能理解一半，后面希望能够回头看看再理解吧，不太擅长
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
