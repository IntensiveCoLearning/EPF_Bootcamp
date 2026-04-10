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
