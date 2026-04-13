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
# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->
今天就正式脱离EL层，进入CL层的学习了

共识协议要解决的核心挑战是：在不可靠的基础设施纸上，构建一个可靠的分布式系统

Ethereum在全球有上万个独立节点，每个节点都维护同一个账本，要求账本必须一致，不是差不多，是必须一致

CL的存在，不是为了加一个新的模块，而是为了让很多彼此不信任的网络条件也一般的节点最终认同一个账本

总结来说就是为了让全网节点对一份链的历史达成一致

共识层到底要解决什么问题？

有很多，比如网络慢，丢包严重，配置错误等等

第一个是自然困难比如网络不稳定，机器有问题，节点掉线等

第二个是对抗性故障比如节点恶意使坏，说谎误导别人

所以共识的问题不是在OP环境下的，而是在节点网络不稳定，还有人捣乱的条件下，怎么样让系统输出一个大家都承认的结果

共识真正解决的是：顺序一致性+结果一致性

Byzantine Fault Tolerance (BFT) 拜占庭容错

这个很经典的问题了

在Ethereum的角度来看，就是：节点还能链接，但是它在胡说，或者发的消息前后不一致

BFT的要求是，就算系统里有一部分这种恶意节点，整个系统仍旧有办法走向正确的共识

这就是为什么BFT在区块链种特别重要的原因

看看 Introduction to Consensus Layer 这一段

这一段把前面抽象的问题，正式落到CL上

Ethereum里的共识具体是在协调谁，按照什么节奏，围绕什么对象达成一致

参与者：nodes，validators

时间节奏：slots、epochs

共识对象：blocks、attestations

CL的目标不是单纯让大家联网，而是让所有诚实节点最终对同一份交易历史和结果达成一致

这个历史是指：交易的发生顺序，最终余额，哪些区块被承认

CL解决的就算全网单一历史的问题

Ethereum 的共识协议，其实是把两个不同协议“拼接”在一起，一个叫 LMD GHOST，一个叫 Casper FFG，它们的组合叫 Gasper

不是一个单一的算法，而是两个算法合作一起工作

我自己的学习笔记中有这一段，就不再多说了

总的来说：

LMD GHOST 更偏“当前链头怎么选”

Casper FFG 更偏“检查点怎么 justify / finalize”

PoW 和 PoS 经常被错误地叫做 consensus protocols，但他们本身并不是共识协议，他们是支持共识协议运行的机制

真正的共识规则是像 LMD GHOST、Casper FFG 这样的协议，PoW / PoS 更像是它们赖以工作的权重与参与约束机制

PoW 和 PoS 主要扮演的是 Sybil resistance mechanisms，他们会给协议设置成本，这样攻击者就不能廉价的攻击系统

这里的sybil经常被使用在空投中，指一个人多个账户

其实项目方才是最大的sybil，不说了

PoW / PoS 很重要，但它们重要的方式是给共识提供权重基础，不是自己单独构成完整共识。

他俩是基础的部分，它们上面还可以搭不同具体协议。

接下来是讲Blockchains

blockchain 的基本元素是 block，一个 block 包含一组由 leader（block proposer）组装的 transactions，block 的具体内容会随着协议不同而不同

区块链区块链，现有区块后有链

区块是区块链的最小结构单元

但是block是通用骨架，block里面装什么，不同阶段和不同链层可以不一样，是可以不是必须

每个块都连着父块，一个接一个形成了链

链随着节点把新区块加上去而增长，这需要暂时选出一个leader负责扩展链

POW中，leader是首先解出puzzle的miner，但是POS中proposer是从 active validator set 里伪随机选出来的

leader会加一个block到链上，然后他要负责选择和排序block里的内容，而且它只接受有效的block

接下来我们进入 Transition to Proof-of-Stake

这一节主要讲的是POW转型POS的历史转变，也就是Merge

Ethereum 不是重启一个新链来改共识，而是在原来飞行中的系统上，直接更换了共识引擎。

Paris hard fork，也就是 The Merge，它不是按 block height 触发的，而是按 terminal total difficulty (TTD) 触发的，这样做是为了避免恶意分叉等风险，只有当累计难度达到某个关键阈值时，才切换到 PoS

系统怎么知道“现在该正式从 PoW 切到 PoS 了”？不是看区块高度，而是看总难度有没有达到终端阈值。

PoW 链的结束点，不是随便挑一个块号，而是“累计工作量跨过某个门槛”的那个点。

Merge引入了什么变化？

1.Beacon Chain 此前已经和 Ethereum mainnet 并行运行。Merge之后它接手处理新区块的职责，在Pos下，出块和验证由validators完成，不再是miners解题

[2.Security](http://2.Security) and Efficiency 用stake替代算力，降低能源成本，让共识机制向可扩展，可持续的方向前进

[3.New](http://3.New) Consensus Mechanism 切到 PoS 之后，共识不是靠“谁先算出来”推进，而是靠一套 validator 协作机制推进。

下面看看 Beacon Chain这套POS共识系统是由哪些部件组成的

1.Validators

POS协议里的参与者，1负责propose block、validate block

成为validator需要向官方质押最少32ETH，最大只能质押2048ETH，如果瞎搞或者失职，会被扣除部分ETH作为惩罚

PoS 不是所有 validator 每个时刻都做同一件事，而是被有组织地分配到不同角色和小组里。

2.Slots and Epochs

一个 slot = 12 秒

一个 epoch = 32 个 slots

所以一个 epoch = 384 秒，也就是 6.4 分钟

每个slot都会分配一个validator来propose block，同时会有 validator committees 去 attest 这个 block

slot是POS共识里最小节拍单位，每12秒系统给一次出块机会

3.Attestations

attestation 是什么？attestation = validator 对链状态/区块有效性的投票。它不是普通消息，而是共识核心输入。

4.Committees

committee 是分配到某个 slot 的 validator 小组，每个 committee 至少 128 validators

committee 最核心的作用有两个：

分工：不是让所有 validator 在每个 slot 都全量做同一件事

安全：把 validator 随机打散进不同小组，提高攻击者集中控制局部投票的难度

下面来讲讲blobs

EIP-4844，也叫 proto-danksharding，属于 Deneb/Cancun hardfork 的一部分

它给 Ethereum 引入了一层数据可用性层，可以临时存储任意数据，这些数据叫 blobs，每个 block 可以带 3 到 6 个 blob sidecars

这是Ethereum向分片或者可扩展性迈出的第一步

blob 不是传统 execution payload 的一部分，而是为数据可用性引入的一种新载体。

EIP-4844 的一个关键设计，是用 KZG commitments 来验证 blobs

链上不会直接把 blob 全量塞进 block header/body；链上主要保存的是对 blob 的 commitment。

后面讲 Checkpoints and Finality

checkpoint 是POS里面专门拿来做更高层确认和finality推进的锚点

block 如果得到多数 validator 的 attestation，就能成为 checkpoint 相关共识的一部分，checkpoint 用来 finalized blockchain state，一个 block 在最近 checkpoint attestations 中拿到 2/3 支持，就能进入 finality 过程

最后讲一些奖惩机制，就不多说了
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->

今天学习 JSON-RPC

JSON-RPC 是一个 remote procedure call protocol，编码格式是JSON，也用来在远端服务器上调用函数并拿回结果

可以把它理解为一种远端调用接口格式，客户端不是直接进入节点内部调用函数，而是发一段JSON请求，然后告诉节点要调用哪个方法，给什么参数，然后把结果按照统一格式返回

执行层客户端对外暴露了一组标准方法，JSON-RPC就是这些方法最常见的承载形式

普通用户和钱包，dapp通常都是通过JSON-RPC和网络交互

共识层和执行层之间，也会通过一种JSON-RPC接口交互，也就是Engine API

一句话来说：JSON-RPC 是执行层把“节点能力”暴露给外部使用者的一种标准接口形式。

接下来看JSON-RPC的请求格式

{

"id": 1,

"jsonrpc": "2.0",

"method": "<prefix\_methodName>",

"params": \[...\]

}

虽然不同方法功能不同，但是它们在长相上都应该一致

id：请求唯一标识

jsonrpc：协议版本

"<prefix\_methodName>"：这个是方法名，但是不是直接的名字，是命名空间前缀+下划线+方法名

params：传给方法的参数列表，没有参数可以穿空数组，有些方法有默认参数

然后下面是 Namespaces

JSON-rpc 的每个方法名，都由两部分组成：

一个namespce prefix和一个具体的方法名，中间用下划线连起来，像：eth\_blockNumber这种

这不是随便起的名字，这是告诉你这个方法属于哪一类能力，大概是干啥的

所以namespace的作用，本质上就是按功能分组

以太坊客户端必须实现规范要求的最小RPC方法几何，但在这个之上，不同客户端还可能提供各自独有的方法和空间名

比如Geth和Reth，不同客户端的namspace和可用方法确实不一样

列了一张表，我们简单看几个

eth：用来与Ethereum本身交互，最常用和最核心的一组

web3：提供一些web3客户端的工具函数

net：提供节点网络信息

txpool：查看交易池

等等吧

先讲 Eth namespace

这是最常用的namespace，用途非常多，比如钱包读取余额，应用查找区块等

具体看一下例子

eth\_blockNumber

没有必需参数

返回最新区块号

eth\_call

参数是 transaction object

立即执行一次 message call，但不会真的把交易上链

这个很重要，它通常用来“模拟执行”或“读取合约视图结果”。

eth\_chainId

没有必需参数

返回当前链的 chain id

eth namespace覆盖了最基本的“看链、看账户、看合约、看日志、做调用模拟”这些核心能力

接下来看 Debug namespace

它不像eth那样服务于钱包，它更偏向区块链浏览器，研究等用途

有些 debug 方法计算量很大，节点执行他们会很繁琐

所以公共RPC一般会限制这个namespace

看一下例子

debug\_getBadBlocks

无必需参数

返回客户端最近看到的坏块

debug\_getRawBlock

参数是 block number

返回 RLP 编码后的 block

debug\_getRawHeader

参数是 block number

返回 RLP 编码后的 header

还有Engine namespace

Engine API 和前面那些普通 Ethereum JSON-RPC 方法不一样。

它不是用户 API，它的唯一目的是让CL和EL交换信息

它跑在单独的，经过认证的endpoint上，不是普通的 HTTP Json-rpc端口，而且使用了JWT认证

一些核心方法：

engine\_exchangeTransitionConfigurationV1

交换 CL 和 EL 的配置细节

engine\_forkchoiceUpdatedV1\*

参数是 forkchoice\_state 和 payload attributes

更新 forkchoice 状态，并可选地触发 payload building

engine\_getPayloadBodiesByHashV1\*

给一组 block hash

返回对应 execution payload bodies

现在可以理解了，前缀就是namespace

下面说encoding

其实就是 JSON-RPC 参数和返回值怎么写

写成0x风格的16进制

Transport agnostic

JSON-RPC 只是“调用格式”，它不强绑定某一种传输协议

它可以跑在不同 transport 上，比如：

HTTP

WebSockets / WSS

IPC

最后是讲使用工具，这里不再赘述

我们看下一个页面 EOF，全名 EVM Object Format

一种给EVM字节码用可扩展可版本化的容器格式。合约在部署时做一次性校验

EOF引入了container的概念，把字节码分出更清晰的结构，让EVM更容易解析

EOF 想把“松散的一串 EVM 字节码”升级成“结构化、可版本化、可提前校验的合约容器格式”。

没了
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->


今天来看看 Data Structures in Execution Layer 页面

首先讲到了RLP

RPL就是 Recursive Length Prefix

它是一种给数据加长度标签的编码方法

解决的问题非常朴素

以太坊里有字符串/字节串和列表，这些东西要发到网络上，写进区块，算hash值，那就必须先变成一段唯一的，确定的，谁都能还原的字节序列

所以RLP不是压缩，也不是加密，也不是hash，他只是序列化格式

可以把RLP当作搬家打包

如果是单个字节串，那就在前面写清楚这段多长

如果一个东西是列表，那就再前面写清楚整个列表多长

如果列表里面还有列表，那就递归的弄

所以它叫 Recursive

那它到底编码什么东西：字节串/列表

注意不是整数，字符串等，这些更高层的东西要先转为字节串，在交给RLP

所以在执行层里面，很多对象先RLP编码，再放进Trie，或者算keccak256

接下来讲的是Trie结构

Merkle Tree 负责“可验证”

Patricia Tree 负责“高效存取”

Merkle Patricia Trie 把两者合在一起

Merkle Tree

Merkle Tree是一种基于hash的树形数据结构

他的核心结构很简单，叶子节点放真实数据，非叶子节点放字节点hash，最后得到顶上的hash，就是Root Hash

Patricia Tree

也叫Radix tree

它不是拿来做hash校验，二十拿来做高效存储和查找的

它不一定是二叉树，边上可以直接标记一串字符，每层不是只能标记一个

Merkle Patricia Trie

以太坊执行层最主要的数据结构就是 Merkle Patricia Trie，简称 MPT

Merkle 给它“加密可验证性”

Patricia 给它“前缀组织和高效查找”

Trie 表示它本质上还是按 key 路径走的树

MPT 为什么既能查又能验

这就是他和普通Patricia trie 的差别

每个节点都有hash值，节点内容先编码，再hash，子节点的引用会影响父节点的hash，所以下层的任意变化都会传到root hash

Patricia 部分保证路径查找高效

Merkle 部分保证根哈希能代表整棵树

没毛病

好了。看接下来的部分吧

下面就是在讲

以太坊里面，到底哪些数据放进MPT？

可以把执行层的数据分类两种

和区块绑定的：Transaction Trie，Receipt Trie

表示全局状态的：World state Trie，每个合约下面还有一个 Storage Trie

先讲 Transaction Trie

他的作用就是 存某一个区块里的全部交易

每个区块都有自己独立的一颗transaction trie

这个区块定了，这颗trie就不能改了

它怎么存？

原文给了最核心的一行：

key 是交易在区块中的 index

value 是交易本身 T

两边都要先 RLP 编码

RLP(index) -> RLP(T)

这点非常重要，它不是拿交易hash来当key，它是拿这笔交易在区块中的为位置当Key

这意味着你如果知道一笔交易在块里的序号，你就能沿着trie找到它

接下来是 Receipt Trie

他也是每个区块一个，叶子里装着与交易一一对应的RLP数据，根写进区块头里的 receiptsRoot

它存的不是“交易本身”，而是“交易执行后的结果摘要”。

核心用途是证明一笔交易确实被执行过，记录结果方便历史索引和验证

还有一个很实际的用途，就是前几天学的 snap sync

全节点同步历史时，不必重放每一笔旧交易来重新生成收据，可以下载区块体里的交易和 receipts，本地重建 receipt trie，然后拿重建出来的根去对区块头里的 receiptsRoot，如果根对得上，就说明这些历史执行结果是可信的

然后讲 World State Trie

这是以太坊最核心的数据结构

它存的不是某个区块里的项目列表，而是“此刻全网所有账户的当前状态”

key：keccak-256 之后的 20 字节账户地址

value：该账户状态的 RLP 编码

world state trie 的叶子，不是随便塞一个值，

而是塞一个账户对象：

\[nonce, balance, storageRoot, codeHash\]

World State Trie 本身并不“整棵存进链里”，真正写进每个区块头的是它的 state root

于是 state root 的意义就很大：

它代表某个区块执行完成后的全局状态

任意节点都可以从前一个状态出发，执行该区块交易，再独立算出这个 root

如果算出来和区块头不一致，就说明这个区块状态转移不对

接下来是 Storage Trie

它和 World State Trie 的关系非常紧密。

每个账户叶子里有个 storageRoot，这个 root 指向一棵单独的 MPT，也就是这个账户自己的 Storage Trie。

World State Trie 叶子里不是直接存合约所有变量，它只存 storageRoot，真正的合约持久化状态在另一棵 trie 里

所以这个是树套树的结构

总结：

Transaction Trie：这个区块里“提交了什么”

Receipt Trie：这些交易“执行结果是什么”

World State Trie：执行完以后“全网账户现在是什么状态”

Storage Trie：某个具体合约“内部持久化变量现在是什么状态”  
  
  
  
  
今天继续学一下 DEVP2P

首先说说 TCP和 UDP

UDP是不建立连接，快，但是不保证顺序和可靠性

TCP是先建立连接，慢一点，但是可靠，顺序也有保障

网络执行层分为两部分

discovery stack，负责找 peer

transport stack，负责 peer 之间真正传消息

发现节点，使用UDP，但是交换数据使用TCP

先说说discovery，节点是怎么发现对方的

起点是 bootnodes，也就是一些写死在客户端或规范里的已知节点，新节点刚加入网络时先去找它们

不用想的太复杂，每一个节点都有一个路由表，上面写了我已知的一些节点，新节点先联系bootnode，然后不断像更接近目标的节点询问，扩展自己的邻居表

原文把这个过程讲成一个 PING-PONG game：

1.新节点先给 bootnode 发 PING

2.bootnode 回 PONG

3.如果这轮对上了，新节点就和它建立 bonding

4.然后新节点发 FIND-NEIGHBOURS

5.bootnode 返回一批邻居

6.新节点再去跟这些邻居重复这个过程

discovery 又分为 Discv4 和 Discv5

Discv4已经能做到分布式找peer，但是整体比较朴素

节点身份基于 secp256k1 公私钥，public key 就是 Node ID，距离靠 XOR 计算，维护 Kademlia 风格的 routing table，用 UDP 传消息，先验证 endpoint，再响应，防止放大攻击

Discv5是在Discv4 之上的增强版

加强了加密通信，更强的身份和session机制，更适合扩展，用签名的ENR

Discv4 更简单，偏明文，扩展性有限

Discv5 更安全，有握手，有 topic-based discovery，更适合大网络

讲讲ENR

ENR 全称是 Ethereum Node Record。它是一个节点的介绍卡片

里面介绍他自己，是谁，在哪，如何连接到它，这个我自己的笔记中有介绍，就不赘述

前面一直在讲怎么找到人，后面开始讲找到以后怎么安全说话

使用RLPx

RLPx 是执行层上基于 TCP 的传输协议

它负责：

建连接

做认证

建 session

封装消息帧

在一条连接上复用多个上层协议

这个过程很重要，不是一连上TCP就直接发送区块数据，先要加密，然后再握手
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->




今天学习EVM及后面的知识

这一节首先的是要告诉我们，不要把EVM理解成运行合约的黑盒程序，先把它理解为状态转换器

EVM在处理交易的时候，会改变Ethereum整体的状态。所以Ethereum可以看作一个State machine，状态会随着输入变化

什么是 state machine？

首先用了一个售货机的例子来讲解

一个state machine有三个基本部分

1.State S

2.Input I

3.State transition function Y

公式是 Υ(S, I) -> S’

意思非常简单，S是当前状态，I是输入，S‘ 是下一个状态,Y 是决定怎么变的函数

![Vending machine](https://epf.wiki/images/evm/vending-machine.gif)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-10-1775804744383-image.png)

自动售货机给了三个状态

Idle Selection Dispensing

给了几个输入：Insetcoin，SelectDrink，CollectDrink

这里有两个事

第一，系统不是随便变换的，他由当前状态+输入+规则决定

第二，不是每个输入都真的会改变状态，有可能输入无效，系统保持原样

Ethrerum 怎么映射到这个模型的？

原文说 Ethereum as a whole can be viewed as a transaction-based state machine.

其实就三个部分：

Ethereum整体是一个状态机，他的输入是transcations，他会进入一个新的状态

这个当前的状态叫world state

自动售货机里的 State

对应 Ethereum 的 world state

自动售货机里的 Input

对应 Ethereum 的 transactions

自动售货机里的 Υ

对应 Ethereum 里负责算状态变化的那套机制，说的就是EVM

那么什么是world state？

world state 本质上是一个从 20-byte 地址到 account state 的映射。

也就是 key是地址，value是这个账户的状态

每个account state里又有很多东西

所以当页面说Ethereum状态变化时，不是说一个抽象的东西变化，本质上是说 地址-》账户的这个映射表发生了变化

EVM is the state transition function of the Ethereum state machine.

这是整页最关键的定义之一

它的意思是：

Ethereum 是状态机

当前有一个 world state

输入是 transactions

EVM 就是负责把“当前状态 + 输入”算成“下一个状态”的那部分

还说了有两种账户类型，分别是

External account

Contract account

这个很简单就不赘述了

接下来看 Virtual machine paradigm

这一节说的是这个抽象的东西该如何落地？

我们理解了状态机，下一步就是 implementation，如何在真实的计算机上跑起来

软件要运行，肯定要换成目标处理器能理解的语言，不同的硬件不同而不同

程序写完不能直接跑，受到硬件架构和操作系统环境的约束

真正要解决跨平台问题，得分两步走

第一步：先面向虚拟机，而不是真实的硬件

程序不要一开始就直接编译成某具体cpu的编码，应该先编译成一种中间形式 bytecode

这个东西是给虚拟机看的。相当于一个中间语言

页面把这个中间承接层叫做：

virtual machine

abstraction layer

这两个词连在一起看，意思就很清楚了：虚拟机把上层程序和下层真实硬件隔开。

第二步再由具体平台上的虚拟机吧bytecode变成真实执行

有一个 platform-specific virtual machine，它把 bytecode 翻译成 native code 来执行

这个方案有两个直接好处

portability：bytecode可以在不同平台运行，不需要为每个平台重新编译应用逻辑

abstraction：把底层硬件复杂性和上层软件逻辑隔开，开发者只需要面向一个统一的虚拟机去写程序

看看EVM这一段写的什么

首先把虚拟机 这个抽象落到EVM这个具体对象上面

EVM不是泛泛的虚拟机概念，它是Ethereum里的那个具体实现

上一节说的是为什么需要虚拟机，这一节说的是Ethereum用的那个虚拟机到底是什么样子的

计算机的结构里面，word是CPU一次能处理的固定大小数据单位

EVM的word size就算32 bytes

在EVM里，32字节是一个非常核心的自然单位，有很多地方，比如堆栈，内存读写，都和这个有关

因为 EVM 选了 32-byte 作为 word size，所以很多执行和存储操作天然围绕 256-bit 这个宽度组织。

EVM不是抽象的处理整个世界的所有东西，而是会具体落到账户的code/storage/balance上。

真实的执行不是直修改一个账户，一般都是多账户交易

EVM 的执行上下文可以跨账户传播，复杂交互就是这么来的。

比如一个交易里，可能出现：

一个外部账户发起调用

一个合约读取自己的 storage

再调用另一个合约

再影响多个账户的余额或状态

![EVM anatomy](https://epf.wiki/images/evm/evm-anatomy.jpg)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ccboom/images/2026-04-10-1775808018882-image.png)

在下面讲了OP code，很简单就不说了

最后我们看看 EVM Data Locations

说了EVM执行的时候，数据主要放在四个地方

Stack：最接近opcode，大多数opcode直接在stack顶部取参数放结果

Memory：执行期间的临时工作区

Storage：合约自己的持久状态，写进去会进入world state

Calldata：外部传进来的输入，只能读不能改

这些我都学习过，所以看了一眼就掠过了，在我的仓库有笔记，就不多说了

接着看看 transaction这个页面

看原文的定义：

A transaction is a cryptographically-signed instruction issued by an external account, submitted to an execution client via JSON-RPC and then broadcasted to the entire network using DevP2p.

这句话有四个重点

cryptographically-signed instructio

交易不是一个普通的消息，它是一个经过密码学签名的指令，要求以太坊执行某件事

issued by an external account

交易是由外部账户发起的，发起者是EOA，并不是 CA 自己主动发交易

submitted to an execution client via JSON-RPC

交易不是直接进入区块的，先提交给execution client，提交入口是 JSON-RPC

broadcasted to the entire network using DevP2p

执行客户端受到之后，再通过DevP2P广播到网络

这就是交易的完整生命周期

接下来复习了一下 transaction fields

nonce

gasPrice

gasLimit

to

value

data / init

signature

这些太简单就不赘述了

看一下下面的 Contract creation

页面选了一个非常短的 runtime code，让你看到：

合约创建交易长什么样

init 和 runtime 是什么关系

为什么创建合约时，data 字段放的是 init bytecode

\[00\] PUSH1 06 // Push 06

\[02\] PUSH1 07 // Push 07

\[04\] MUL // Multiply

\[05\] PUSH1 0 // Push 00 (storage address)

\[07\] SSTORE // Store result to storage slot 00

看一下这段汇编 计算 6 \* 7，然后把结果写进 storage slot0

\[00\] PUSH1 06

把 0x06 压到栈顶。

\[02\] PUSH1 07

把 0x07 压到栈顶。

\[04\] MUL

从栈顶取两个值相乘。

结果是 0x2a，也就是十进制 42。

\[05\] PUSH1 0

把 0x00 压栈，表示 storage slot 0。

\[07\] SSTORE

把值写到 storage。

接下来看看init

<init bytecode> <runtime bytecode>

创建合约交易里放进去的，不知有runtime code 还有 init code

init 只在创建合约的时候运行一次，用来生成最终部署的代码

先执行一段 init code

由它“返回”最终要存进去的 runtime code

看一段汇编

// 1. Copy to memory

\[00\] PUSH1 08 // PUSH1 08 (length of our runtime code)

\[02\] PUSH1 0c // PUSH1 0c (offset of the runtime code in init)

\[04\] PUSH1 00 // PUSH1 00 (destination in memory)

\[06\] CODECOPY // Copy code running in current environment to memory

// 2. Return from memory

\[07\] PUSH1 08 // PUSH1 08 (length of return data)

\[09\] PUSH1 00 // PUSH1 00 (memory location to return from)

\[0b\] RETURN // Return the runtime code and halt execution

// 3. Runtime code (8 bytes long)

\[0c\] PUSH1 06

\[0e\] PUSH1 07

\[10\] MUL

\[11\] PUSH1 0

\[13\] SSTORE

先把 runtime bytecode 复制到 memory

然后从 memory 返回 runtime bytecode

这就是 contract creation 的核心示范

现在慢慢看这 6 条关键指令。

\[00\] PUSH1 08

runtime code 长度是 8 bytes。

先把长度压栈。

\[02\] PUSH1 0c

runtime code 在 init 里的起始偏移是 0x0c。

把这个偏移压栈。

\[04\] PUSH1 00

目标 memory 地址设为 0x00。

也就是想从 memory 开头开始放。

\[06\] CODECOPY

把“当前执行环境里的 code”某一段复制到 memory。

页面这里明确想表达的是：

当前执行环境就是 init code 本身

runtime code 就嵌在这段 code 的后半段

所以 CODECOPY 可以把它抽出来放到 memory

然后进入第二步。

\[07\] PUSH1 08

告诉 RETURN：要返回 8 个字节。

\[09\] PUSH1 00

从 memory 的 0x00 开始返回。

\[0b\] RETURN

把 memory 里的这 8 个字节返回，并结束 init 执行。

然后给出transaction payload

\[

"0x", // nonce

"0x77359400", // gasPrice

"0x13880", // gasLimit

"0x", // to

"0x05", // value

"0x6008600c60003960086000f36006600702600055", // init code

\];

nonce = 0x

表示这是该账户第一笔交易。

to = 0x

这里最关键。

to 为空，页面前面已经讲过，这就意味着：

这是一笔 contract creation transaction。

value = 0x05

给新合约发 5 wei

页面给出的返回里，重点看这几个：

status: "0x1"

to: null

contractAddress: "0x5fbd..."

gasUsed

effectiveGasPrice

status = 0x1

说明交易成功。

to = null

符合 contract creation 语义。

因为这不是发给已有地址，而是在创建新地址。

contractAddress

这是最关键的产物。

说明这笔交易创建出来的新合约地址是多少。

这一节的重点是什么

创建合约的时候，to是空的

创建合约交易里放的是 init code

init code只执行一次

RETURN 的是数据会成为新合约账户里的code

value会成为新合约账户的初始余额
<!-- DAILY_CHECKIN_2026-04-10_END -->

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
