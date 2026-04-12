---
timezone: UTC+8
---

# Bein

**GitHub ID:** Minami-Bein

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->
本周主要围绕 **Ethereum 执行层的基础认知** 展开，整体从“概念理解”逐步过渡到“流程理解”，核心内容包括：

-   以太坊整体架构（Execution Layer / Consensus Layer）
    
-   账户体系（EOA 与 Contract Account）
    
-   Gas 机制与交易基础
    
-   EVM 执行模型
    
-   交易结构与执行流程
    
-   节点通信机制（JSON-RPC / DevP2P）
    

通过一周的学习，已初步建立起对以太坊系统的整体认知框架。

相比刚开始的零散知识，本周最大的提升在于：

1\. 从“知识点”到“执行链路”

逐步串联起完整流程：

> 用户构造交易 → 本地签名 → JSON-RPC 提交 → 节点接收 → mempool → P2P 广播 → 区块打包 → EVM 执行 → 状态变更

不再是孤立理解某个模块，而是从整体视角看待系统运作。

* * *

2\. 明确不同层级的职责划分

开始区分不同组件的角色：

-   客户端（钱包 / SDK）：负责构造与签名
    
-   节点（Execution Client）：负责验证、传播与执行
    
-   网络层（DevP2P）：负责数据同步与广播
    
-   EVM：负责具体的状态计算
    

这种分层理解让系统结构更加清晰。

* * *

3\. 对“链上 vs 链下”有初步认知

开始意识到：

-   很多逻辑是在链下完成（如签名、交易构造）
    
-   链上只负责验证与状态执行
    

这为后续理解 **Account Abstraction（账户抽象）** 打下基础。

这一周逐渐体会到：

-   区块链系统本质是一个“分层协作系统”，而不是单一技术点
    
-   每一笔交易背后涉及客户端、网络、执行层的协同工作
    
-   学习过程中不能只停留在 API 或工具使用，需要理解底层机制
    

同时也意识到，后续需要从“理解流程”进一步走向“理解实现”。
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->

今天重点学习了 **Ethereum 网络通信机制**，主要包括：

-   DevP2P 协议（节点之间的 P2P 通信机制）
    
-   交易与区块在网络中的传播方式
    
-   JSON-RPC 接口的实际作用（客户端与节点交互）
    
-   节点在执行层中的角色划分
    

通过这一部分的学习，对“交易是如何从用户传播到全网”的过程有了更清晰的认知。  
从整体流程角度，对交易广播过程进行了梳理：

1.  用户通过钱包或 SDK 构造交易
    
2.  使用 JSON-RPC 将交易发送到某个节点
    
3.  节点验证交易合法性后放入本地 mempool
    
4.  通过 DevP2P 协议，将交易广播给其他节点
    
5.  网络中节点逐步同步交易数据，最终被矿工/验证者打包
    

重点理解了两个关键点：

-   **JSON-RPC 是入口（用户 → 节点）**
    
-   **DevP2P 是传播（节点 → 节点）**
    

同时对比发现：

-   JSON-RPC 更偏“应用层接口”
    

DevP2P 更偏“底层网络协议”  
今天的核心收获是把“交易执行”与“交易传播”这两个过程区分开来：

-   之前更关注交易如何执行（EVM 层）
    
-   今天开始理解交易如何在网络中流动（Network 层）
    

逐渐建立起一个更完整的链路认知：

> 用户 → JSON-RPC → 节点 → P2P 网络 → 全网传播 → 区块打包 → EVM 执行

同时也意识到：

-   网络层的效率直接影响交易确认速度
    
-   客户端节点不仅是执行者，也是网络参与者
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->


今天主要围绕 **Ethereum 交易结构与执行流程** 进行学习，重点包括：

-   Transaction 的核心字段（nonce / gas / to / value / data）
    
-   RLP Serialization（交易编码规则）
    
-   JSON-RPC 接口在交易发送中的作用
    
-   Execution Layer（执行层）在交易处理中的职责
    

PS：通过阅读文档，对交易从构造到上链的整体流程有了更清晰的认识。

  
在理解基础概念的同时，重点从“流程角度”进行了梳理：

1.  用户侧构造交易（或通过 SDK 封装）
    
2.  交易进行签名（本地完成）
    
3.  通过 JSON-RPC 将交易发送到节点
    
4.  节点接收后进入 mempool（交易池）
    
5.  被打包进区块后进入 EVM 执行
    

同时尝试从代码/SDK角度去理解：

-   哪些逻辑是在客户端完成（如签名、参数构造）
    
-   哪些逻辑是在链上执行（如状态变更）
    

遇到的问题：

-   RLP 编码规则相对抽象，目前更多停留在理解层面，缺少实际编码体验
    
-   EVM 执行过程较复杂，特别是 gas 消耗与 opcode 执行细节
    
-   对客户端与执行层之间的边界理解还不够清晰
    

思考与收获

今天最大的收获是开始从“知识点”转向“执行流程”的理解：

-   交易不仅是一个数据结构，更是一个完整的执行链路
    
-   客户端负责构造与提交，链上负责验证与执行，这种职责划分非常清晰
    
-   对后续学习 Account Abstraction 有了铺垫：  
    用户操作本质上可以被重新抽象，而不仅仅是传统交易
    

明日计划

-   继续深入 P2P 网络与 JSON-RPC 的具体实现方式
    
-   进一步理解节点之间如何传播交易
    
-   尝试结合 SDK 或示例代码，加深对交易发送流程的理解
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->



![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/Minami-Bein/images/2026-04-08-1775669487734-image.png)

维修bug
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->




![77170d6b7c728564014b6c86870cb6aa.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/Minami-Bein/images/2026-04-07-1775578871375-77170d6b7c728564014b6c86870cb6aa.png)

学习密码学
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->





![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/Minami-Bein/images/2026-04-06-1775491855322-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/Minami-Bein/images/2026-04-06-1775492614139-image.png)
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->






第一天
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
