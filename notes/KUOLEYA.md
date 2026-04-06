---
timezone: UTC+8
---

# KUOLEYA

**GitHub ID:** KUOLEYA

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->
今天主要学习了第一周课程的The Protocol部分

其中Prehistory部分讲述了以太坊的发展历程它代表的是根植于**早期互联网的开放精神**、**Unix的简洁哲学**、**自由开源软件运动**、**密码朋克的隐私抗争**，以及**比特币的去中心化货币实验**

[Architecture部分：](https://epf.wiki/#/wiki/protocol/architecture)介绍了协议是由两个部分——执行层（EL）和共识层（CL）两部分组成

EL负责实际处理用户端发出的交易请求 CL负责监管和验证节点是否遵守规范，

并且EL和CL各自连接不同的P2P网络来获取“原材料”（交易和区块），并通过API进行两个层级之间的数据传输

[Design rationale：](https://epf.wiki/#/wiki/protocol/design-rationale)讲述了以太坊自诞生之初的设计理念：**简洁性**，**普遍性，模块化，非歧视性，敏捷性**：

而协议的设计理念需要遵守如下的原则

-   **满足需求的前提下，协议应尽可能简单。** 当复杂性不可避免时，遵循"夹心模型"——让底层（核心共识）和顶层（用户接口）保持简单，将复杂性推入协议的"中间层"。
    
-   **宁可选择内部复杂但有清晰外部接口的子系统，也不选择各部分纠缠不清的系统。**
    
-   **用户使用协议的目的不应受到限制（网络中立性原则）。**
    
-   **协议功能和操作码应代表尽可能底层的概念，以便能以任意方式组合。**
    
-   **拒绝将常见的高层次用例直接纳入协议。** 如果用户真的需要，完全可以在合约层面通过底层功能模拟实现。
    

**在最后的**[**Evolution**](https://epf.wiki/#/wiki/protocol/history)**部分讲述了以太坊从发布之初的几个重要节点以及其引入的一些特色协议**
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
