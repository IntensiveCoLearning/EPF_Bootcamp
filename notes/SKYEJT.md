---
timezone: UTC+8
---

# SKYEJT

**GitHub ID:** SKYEJT

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
今天开始真正从EL层面开始入门

作为以太坊执行层（Execution Layer）的核心工作，状态转移函数其实只需要回答两个最根本的问题**1**：

1.  这个新打包好的区块，到底能不能合法地添加到区块链的末尾？
    
2.  如果把它加进去了，大家账户里的钱和数据（即“状态”）会发生什么变化？
    

为了让你更好理解，官方资料里用了一个非常经典的数学公式来概括这个过程：_σt_+1​≡Π(_σt_​,_B_)。 别被公式吓到，我们把它翻译成大白话就是：**新状态 = 运作规则(旧状态, 新区块)**。

-   _σt_​**（旧状态）**：代表在处理新交易**之前**，那个“超级大账本”当前的模样（比如张三有5块钱，李四有3块钱）。
    
-   _B_**（新区块）**：代表一个装满最新交易的“包裹”（比如里面有一笔记录：张三给李四转了2块钱）。
    
-   Π**（状态转移函数）**：代表以太坊的“运作规则”或处理引擎。它负责打开这个包裹，按照既定规则处理里面的每一笔交易。
    
-   _σt_+1​**（新状态）**：代表处理完包裹里所有交易**之后**，“超级大账本”更新后的最新模样（变成张三剩3块钱，李四变成5块钱）。
    

需要注意的是，以太坊的“状态”并不是像一个文本文档那样保存在某个具体的文件夹里，而是通过运行这些交易后**动态计算推导**出来的。

**那么，这个“运作规则”具体是如何一步步干活的呢？** 根据执行层的代码规范，处理一个新区块（执行状态转移函数）就像是工厂流水线上的质检加加工过程，主要包含以下几个关键步骤：

1.  **核对历史记录**：首先找出区块链上的上一个区块（父区块），看看账本目前处于什么进度。
    
2.  **严格验明正身（Header Validation）**：检查新区块的“区块头”（Header）。比如要核对它附加的数据（如 blob gas）对不对，并将它的参数与父区块进行对比验证，确保它没有伪造或违反基本规则。
    
3.  **清空多余数据**：检查并确保区块中名为 ommers（以前叫叔区块）的字段是空的。
    
4.  **逐一执行交易（Block Execution）**：这是最核心的一步。系统会把区块里打包的交易一笔一笔地执行。执行完后，系统会得出几个结果：这批交易总共消耗了多少 Gas（手续费）、所有交易的最终日志（Logs Bloom）、所有数据的“根目录”（Trie Roots），以及最重要的——**执行完所有交易后的“新状态（State）”**。
    
5.  **最终复核**：系统会把自己刚刚算出来的“新状态”的根哈希值等参数，跟新区块自带的摘要信息进行对比。如果两者完美吻合，说明计算无误。
    
6.  **盖章入库**：如果上述所有检查和计算都顺利通过，这个新区块就会被正式挂到区块链上。
    
7.  **如果发现错误**：在任何一个环节（比如某笔交易不合法，或者算出来的结果对不上），系统就会直接报错抛出“无效区块（Invalid Block）”，并拒绝更新账本。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->

请假一天
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


接着昨天的底层协议部分，总结了账户模型和UTXO，还有关于状态存储的MPT-VERKLE 还有数据序列化RLP-SSZ（还了解了什么是默克尔化）等等，还总结了协议的历史与演变（但是里面涉及到还有很多具体的名词和实际操作还不熟悉）

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/SKYEJT/images/2026-04-08-1775656787655-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/SKYEJT/images/2026-04-08-1775656239145-image.png)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->



以太坊的设计核心理念：简洁性、通用性、模块化、非歧视性、敏捷性。还有关于一些原则的描述。现在来看显得比较空洞，再进一步学习区块链层协议

1.  了解到新名词UTXO和Accounts总结了两个模型对比：（具体的内容笔记在我自己的NOTEBOOKLM）
    

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/SKYEJT/images/2026-04-07-1775542567370-image.png)
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->




第一天看完了[Prehistory（史前历史）](https://epf.wiki/#/wiki/protocol/prehistory)（以太坊起源与背景），让我回想起了这个经典的RSA加密算法，以公因数分解为背景的非对称加密算法（曾经的密码学知识开始苏醒哈哈哈哈）

再看到free as in freedom 这里不禁感慨果然还是只有开源才能促进发展。和我们科研一样只有开源代码才是真正有价值的科研。

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/SKYEJT/images/2026-04-06-1775485164941-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/SKYEJT/images/2026-04-06-1775485341438-image.png)

可能是小白，现在看这两张流程图还是有点一头雾水。不清楚里面具体执行层和共识层是怎么样的，以及这个通过API是如何调用的。慢慢学习把！
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
