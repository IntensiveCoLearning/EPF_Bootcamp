---
timezone: UTC+8
---

# wLynna

**GitHub ID:** wLynna

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
04/07

## 🧩 核心结构（Core Structure）

以太坊被拆分为两层：  
**执行层（Execution Layer, EL）**  
**共识层（Consensus Layer, CL）**

👉 分离：  
**执行（what happens） vs 共识（what is agreed）**

* * *

## 🧮 执行层（Execution Layer, EL）

所有交易真正发生的地方  
👉 执行逻辑 + 数据变化  
👉 Global Computer

~ **EVM（Ethereum Virtual Machine）**  
执行智能合约的“虚拟计算机”

~ **State（状态）**  
存储所有链上数据（余额、合约）

~ **Transactions（交易池 / mempool）**  
等待被打包的交易

~ **p2p network**  
传播交易与区块数据

* * *

## 🤝 共识层（Consensus Layer, CL）

决定哪条链是“正确的”  
👉 验证 + 选择 + 最终确认  
👉 Global Agreement

~ **Validators（验证者）**  
负责出块和投票（质押ETH）

~ **PoS（Proof of Stake）**  
基于质押的共识机制

~ **Fork Choice（LMD-GHOST）**  
决定当前主链是哪条

~ **Finality（Casper FFG）**  
确定区块不可回滚

~ **RANDAO**  
生成随机数（用于选择验证者）

~ **p2p network**  
传播区块与投票

* * *

## 🔌 层间连接（Layer Communication）

**Engine API（核心桥梁）**

~ 共识层 → 执行层  
“执行这个区块”

~ 执行层 → 共识层  
“执行结果是这样”

👉 Execution ↔ Consensus

* * *

## 🌐 对外接口（Interfaces）

~ **JSON-RPC API**  
用户 / dApp → 执行层  
（发送交易、调用合约）

~ **Beacon API**  
验证者 → 共识层  
（参与共识）

* * *

## 🧠 三层极简模型（Mental Model）

| 层 | 作用 | 关键词 |
| --- | --- | --- |
| 🧮 Execution | 做事情 | 执行 |
| 🤝 Consensus | 定规则 | 共识 |
| 🔌 Interface | 连接外部 | 入口 |

* * *

## ⚡ 一句话总结

**Ethereum = Execution + Consensus + Communication**

以太坊 = 执行 + 共识 + 连接
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->

2026/04/06

今天学习两部分

首先搞明白 如何学习，在哪里打卡

其次快速学习 以太坊 史前历史

## 《Prehistory of Ethereum》

**Ethereum is not an isolated invention — it is the natural outcome of decades of open-source culture, cryptography, and cypherpunk ideology.  
以太坊不是一个孤立的发明，而是几十年开源文化、密码学发展与赛博朋克思想的自然结果.**

### 五条核心演进路径

1.  Internet（互联网）
    
2.  Unix & Open Source（开源文化）
    
3.  Cryptography（密码学）
    
4.  Cypherpunk Movement（赛博朋克）
    
5.  Digital Currency Experiments（数字货币尝试）
    

👉 最终汇聚 → Ethereum

### Ethereum = 技术 × 文化 × 思想 × 实验 的交汇点

1️⃣技术层 Technology ：Internet Unix Cryptography，提供“能力”  
2️⃣ 文化层（Culture）： Open Source、Hacker Culture；提供“协作方式”  
3️⃣ 思想层（Ideology）：Cypherpunk，Freedom / Privacy；提供“价值观”  
4️⃣ 实验层（Experiments）：DigiCash，RSA / PGP；提供“试错路径”

### Key Insights：

Ethereum is an evolution, not an invention.  
以太坊是演化出来的，不是凭空创造的。

Cryptography made trust optional.  
密码学让“信任”变成可选项。

Open systems beat closed systems in the long run.  
开放系统最终胜过封闭系统。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
