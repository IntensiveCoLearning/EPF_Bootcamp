---
timezone: UTC+8
---

# futureclover87

**GitHub ID:** futureclover87

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->
学习了 ETH 共识层 原理
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->

今天学习了下以太坊Layer 2 扩容链 和EVM 兼容 Layer 1公链
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->


ETH Garden Lv12 close
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->



ETH 关卡挑战到LV9
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->




参加EPF周会 & Infinite Garden
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->





Day1-协议学习

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/futureclover87/images/2026-04-06-1775481587646-image.png)

UTXO-Unspent Transaction Output 未花费交易输出

不可分割性

比特币使用 UTXO，而以太坊（Ethereum）使用账户模型。两者的区别可以用\*\*“钱包里的现金”和“银行卡数字”

UTXO优点：

安全性高： 每一笔钱都能追溯到最初的矿工奖励，极难造假。

高并发性： 如果你有 10 张 100 元，你可以同时在 10 个不同的商店买东西，互不干扰。在区块链中，只要涉及的是不同的 UTXO，交易就可以并行验证。

隐私保护： 用户可以为每一次找零生成新的地址，从而隐藏资产总额。

缺点：

开发难度： 对于开发者来说，处理“找零”和“碎片整合”逻辑比直接加减余额要复杂。

状态爆炸： 随着时间推移，网络中会出现大量极其微小的 UTXO（尘迹交易），占用节点存储空间。

协议复杂性设计

三明治模型复杂性 (Sandwich Complexity)

1）底层（底层协议/共识层）： 必须保持极其简单。像比特币一样，逻辑越少，漏洞越少，攻击面越窄。

2）顶层（应用层）： 必须保持极其简单。对于开发者和用户来说，操作应该是直观的。

3）中间层（封装/转换层）： 所有的复杂性都被“塞”进了这一层。

三明治模型通过牺牲“中间层”的简洁性，换取了底层协议的稳固和顶层应用的多样化。

封装式复杂性 (Encapsulated Complexity)

“封装”是工程学中的常用手段，但在区块链设计中，它指的是一个功能是内置在协议底层（Enshrined），还是放在协议之上。

1）低封装（模块化）： 协议只提供最基础的积木（如哈希函数、签名验证），剩下的由二层网络（L2）或合约实现。优点是底层极其干净，缺点是系统整体效率低，用户体验割裂。

2）高封装（封装式）： 将某些常用功能直接写死在以太坊的底层代码里（例如：将 ZK-Proof 验证或特定的账户抽象逻辑直接内置）。

模块化路线图 (Modular Roadmap)

以太坊不再试图在一条链上完成所有事。

执行交给 L2（处理复杂计算）。

结算和数据可用性留给底层 L1（处理简单、高安全的共识）。 这种分离就是一种空间上的复杂性隔离。

渐进式封装 (Progressive Enshrinement)

以太坊官方非常谨慎。只有当某个功能（如账户抽象 EIP-4337）在应用层被证明是极其必要且已经成熟时，他们才会考虑通过 EIP（以太坊改进提案）将其部分“封装”进协议层。

账户抽象 (Account Abstraction)。它通过把“签名校验”和“资产所有权”解耦，把原本需要用户自己处理的复杂逻辑（如保存私钥、支付 Gas）封装进智能合约里。对用户来说，复杂性消失了；但在底层，复杂性被转化成了代码逻辑。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
