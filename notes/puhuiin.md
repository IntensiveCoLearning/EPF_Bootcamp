---
timezone: UTC+8
---

# puhuiin

**GitHub ID:** puhuiin

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->
今天学习以太坊的提议者-构建者分离（PBS, Proposer-Builder Separation）机制以及MEV-boost

传统以太坊验证者同时负责选择交易、构建区块和提议区块 。

PBS 将职责分为：构建者。提议者。验证者不

当前的实施方案：MEV-boost，为验证者实现了一个构建者 API，并处理构建者、中继和提议者之间的通信 。
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->

今天学习以太坊路线图与扩展性：pow——》pos——》rollup——》Verkle树——》减负，清理超过一年的旧历史数据——》ERC-4337智能钱包

扩展性：Layer 1负责结算和数据可用性，Layer 2执行
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->


今天主要学习了Ethereum的协议路线图：以太坊并没有一个固定的终极版本，而是一个持续升级的无限花园，通过社区共识不断优化性能、安全和去中心化。

PoW-》PoS

扩展性，Rollup，Proto-Danksharding、完整Danksharding和数据可用性抽样（DAS），本质是通过“链下执行 + 链上验证”来提升吞吐量。

MEV，PBS、MEV Burn机制
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->



今天复习以太坊执行层核心机制，EVM 对象格式升级EOF、内置预编译合约的作用，交易打包，区块构建
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->




复习的共识层 (CL)状态转换函数 Y(S, T) = S，如果失败，状态回滚到S，但 Gas 费照扣

对比下RLP，SSZ

RLP (Recursive Length Prefix)：动态编码，没有类型，省空间，缺点是必须从头开始顺序解码才能找到想要的数据，无法进行结构化验证

SSZ (Simple Serialize)：强类型的，天然支持默克尔化，生成轻客户端证明变得高效
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->





今天学习了分叉选择机制（就是pow，pos），链重组与回滚，节点在接收到新区块或投票时，会使用新信息重新评估分叉选择规则 。安全性与活跃性，安全性意味着不会发生坏事，活跃性意味着好事终将发生。还学了架构与控制流，信标节点，状态转换函数
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->






今天学习弱主观性与同步机制。以太坊合并进入权益证明pos阶段后，引入了弱主观性的概念，改变了节点同步和信任的模式。当验证者退出后，如果大量退出验证者重新认证过去的区块，可能会创建平行的链历史 。
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->







今天学习SSZ,SSZ 是专为信标链设计的序列化方案，弥补了 RLP 的不足，原生支持复杂数据结构，并大幅提升了哈希效率和数据验证的确定性 。
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->








学习简单序列化，SSZ 取代了执行层中使用的RLP序列化方案。SSZ能提升以太坊共识层的效率、安全性和可扩展性。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->










今天学习CL职责与整体流程，拜占庭将军问题和拜占庭容错。对共识层有了基本的了解
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->











今天学习内置加密原语合约和交易打包与出块流程
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->












学习EVM 对象格式，学了一会儿solidity，返回值和修饰符
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->













今天学智能合约入门：Solidity 基础语法，Remix IDE 在线编译 + MetaMask 连接测试网部署，Gas 费
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->














今天学习RLP编码规则DevP2P
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->















今天学习交易字段与生命周期区块与状态相关结构
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
















今天例会，自学学习el客户端以及evm
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->

















学习了以太坊Protocol与Execution Layer的结构
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
