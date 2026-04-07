---
timezone: UTC+0
---

# kvuxnz

**GitHub ID:** kvxunz

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->
\# Ethereum 前史

\> "Heroes are heroes because they are heroic in behavior, not because they won or lost." — Nicholas Taleb

追溯 Ethereum 的思想谱系——从互联网的诞生、Unix 哲学、公钥密码学革命、自由软件运动、密码朋克运动、数字货币实验，到 Bitcoin 和 Ethereum 的诞生。

\---

\## 一、信息高速公路：互联网的崛起

\- **1969**：互联网前身 \[\[ARPANET\]\] 作为冷战项目诞生

\- 广播用了 38 年才触及 5000 万用户，电视 13 年，PC 16 年，而\*\*互联网只用了 4 年\*\*

\- 互联网消除了地理边界，使原本不可想象的人类交互成为可能

\> "National borders are just speed bumps on the information superhighway." — Timothy May

\---

\## 二、Unix 与 Bell Labs

\### 起源

\- Unix 诞生于对 MULTICS（1960 年代庞大操作系统项目）的简化尝试

\- **Ken Thompson** 和 **Dennis Ritchie** 在 AT&T Bell Labs 创造了 Unix——一个更模块化、更简单、更可组合的替代方案

\- Ken Thompson："三周就有了操作系统——一周编辑器，一周汇编器，一周内核"

\- 1972 年 Dennis Ritchie 编写了影响深远的 **C 语言**

\### Bell Labs 的创新

\- Bell Labs 是 20 世纪最重要的技术孵化器

\- 其创新不是直接面向消费者的产品，而是嵌入通信基础设施的\*\*平台级创新\*\*

\- Ethereum 在很多方面就像一个开放的 Bell Labs

\### Unix 哲学

\- 层级文件系统

\- Shell 命令行接口

\- 单一用途工具的组合（做一件事并做好）

\- 简单性、灵活性和可复用性

\- Unix 及其衍生系统（Linux、macOS）至今仍是现代计算的基石

\---

\## 三、密码学革命：我们能保守秘密吗？

\### 问题

自文明诞生以来，密文传递一直是人类的核心需求。传统的\*\*对称加密\*\*（加解密用同一把钥匙）使得密钥分发成为噩梦——二战中 Alan Turing 破解 Enigma 机就是明证。

核心问题：\*\*如何在从未见面的人之间安全交换密钥？\*\*

\### 公钥密码学的诞生（1974-1978）

| 年份 | 人物 | 突破 |

|------|------|------|

| 1974 | **Ralph Merkle** | Merkle's Puzzles——首个无需预共享秘密即可协商密钥的方法 |

| 1976 | **Whitfield Diffie & Martin Hellman** | 发表《New Directions in Cryptography》，提出 Diffie-Hellman 密钥交换，开创\*\*无信任密码学\*\* |

| 1977 | **Ron Rivest, Adi Shamir, Len Adleman** | 发明 RSA 密码系统——首个公钥加密的工程实现 |

\- Martin Gardner 在 Scientific American 发表 RSA 文章，附带 RSA-129 挑战（悬赏 $100）

\- 1994 年该挑战被破解，奖金捐给了自由软件基金会

\- 1997 年英国政府解密了 1970 年的类似研究（英国情报机构更早独立发现了公钥密码学）

\### 影响

现代 RSA 加密（1024-4096 位）为信息高速公路创造了安全通道，使银行和信用卡公司能保护金融交易，推动了电子商务和网上银行的发展。

\---

\## 四、自由软件运动：Free as in Freedom

\### 软件从开放到封闭

\- **1950-60 年代**：早期软件原始且需要修改，源代码共享是常态，形成了"黑客文化"

\- 计算机杂志甚至会刊登 type-in 程序，鼓励用户手写代码

\- **1969**：US vs. IBM 反垄断诉讼后，软件开始独立收费，变成商品

\- Unix 也未能幸免——AT&T 在 1980 年代初停止免费分发

\- Bill Gates 发表致爱好者的公开信，要求停止共享 BASIC 源代码

\### Richard Stallman 与 GNU

\- MIT AI 实验室研究员 **Richard Stallman** 因无法修改 Xerox 打印机源代码而愤怒

\- 他认为限制软件修改是"对人类的犯罪"

\- **1983**：通过邮件宣布 GNU 项目（GNU's Not Unix，递归缩写）

\- **1984**：GNU 正式启动，编写了 GPL（GNU 通用公共许可证）

\- "Free software" 指的是\*\*自由\*\*（free speech），不是\*\*免费\*\*（free beer）

\- 到 1990 年，GNU 完成了操作系统的所有主要组件，唯独缺少\*\*内核\*\*

\### Linus Torvalds 与 Linux

\- **1991**：芬兰计算机科学学生 **Linus Torvalds** 开发了 Linux 内核

\- 发布后数小时内就收到响应，一年内数百人加入开发

\- Linux 以 GPL 许可发布，补全了 GNU/Linux 操作系统

\- Linux 奠定了基于\*\*社会共识\*\*的软件开发蓝图

\### 开源运动

\- 开源运动与自由软件运动有别，更侧重源代码可访问的\*\*实际收益\*\*

\- 在社区驱动创新与商业可行性之间取得平衡

\- FOSS（Free and Open-Source Software）是两者的总称

\---

\## 五、密码朋克写代码

\### 密码战争

\- 二战后，各国政府垄断密码学进展

\- 美国将加密技术列入\*\*军火管制清单\*\*，NSA 对密码学研究高度敏感

\- NSA 曾试图阻止 RSA 论文发表（最终允许）

\- NSA 雇员 Joseph Mayer 写信给 IEEE 称密码学出版物需政府批准，引发公众强烈批评

\### PGP 与 Phil Zimmermann

\- **1991**：\*\*Phil Zimmermann\*\* 开发了 PGP（Pretty Good Privacy），让个人也能使用强加密

\- **1993**：因涉嫌违反加密软件出口限制，遭美国海关刑事调查

\- 他将 PGP 完整源代码出版为\*\*精装书\*\*，主张图书出口受第一修正案保护

\- 欧洲志愿者扫描书页并 OCR 还原为电子版（超过 70 人工作超 1000 小时）

\- **1996**：案件撤销

\### 密码朋克运动

\- **1992**：\*\*Eric Hughes\*\*、\*\*Timothy C. May\*\*、\*\*John Gilmore\*\* 创立 Cypherpunk 邮件列表

\- 超过 **700 名活动家和反叛者**（包括 Zimmermann）

\- 核心信条：

\- 政府不应窥探公民事务

\- 保护通信和交换是基本权利

\- 这些权利需要通过\*\*技术\*\*而非法律来保障

\- 技术的力量创造新的政治现实

\> "Cypherpunks write code."

\### 加密无政府主义

\- **1988**：Tim May 发表《加密无政府主义宣言》

\- 反对一切权威形式，只承认密码学描述和代码执行的法则

\- 设想匿名数字交易是个人自由的基石

\- **缺失的拼图：一种密码学原生的数字货币**

\---

\## 六、寻找缺失的拼图：数字货币实验

| 年份 | 项目 | 创建者 | 特点 | 结局 |

|------|------|--------|------|------|

| 1990 | **DigiCash** | David Chaum | 首个匿名数字经济体验；依赖现有金融基础设施，高度中心化 | 1998 年破产 |

| 1996 | **E-gold** | — | 以实物黄金为储备；峰值 350 万注册账户 | 2009 年因法律问题暂停 |

| 1998 | **B-money** | Wei Dai | 用密码学函数创造货币，去中心化设计 | 未落地 |

| 2005 | **BitGold** | Nick Szabo | 数字化控制稀缺性，去中心化设计 | 未实现 |

B-money 和 BitGold 虽未成功，但其设计直接影响了 Bitcoin。

\---

\## 七、Bitcoin

\- **2008 年金融危机**重新点燃了对数字货币的兴趣

\- **Satoshi Nakamoto**（化名）发表论文《Bitcoin: A Peer-to-Peer Electronic Cash System》

\- 解决了\*\*无领导者共识\*\*这一开放性问题

\- Bitcoin 的核心创新：

\- 分布式账本：数据以密码学方式按时间顺序链接成区块

\- 首个去中心化数字货币：无底层抵押品，无需银行等可信第三方

\- 工作量证明（Proof of Work）区块链：公开就交易顺序达成共识

\> Vitalik Buterin："Satoshi 同时引入了两个激进且未经验证的概念：作为货币的 bitcoin，以及基于工作量证明的区块链。"

Bitcoin 网络上也有人尝试构建应用（Colored Coins、Namecoin），但 Bitcoin 的网络对此过于原始，应用只能通过复杂且不可扩展的变通方案实现。

\---

\## 八、Ethereum 世界计算机

\### 诞生

\- **2012**：\*\*Vitalik Buterin\*\* 与 Mihai Alisie 创办 Bitcoin Magazine

\- Vitalik 很快发现 Bitcoin 的局限性，提出支持\*\*通用金融应用\*\*的平台

\- **2014**：在 **Gavin Wood** 的帮助下，Ethereum 的设计被形式化（Yellow Paper）

\- **2015 年 7 月 30 日**：Ethereum 主网上线

\### 定位

\- 不仅是数字货币，而是构建\*\*自主经济工具\*\*的平台

\- 支持智能合约和去中心化应用（dApps）

\- 截至原文撰写时，市值约 **$400 billion**

\### 谱系总结

\`\`\`

互联网 (1969) → Unix/Bell Labs (1970s) → 公钥密码学 (1974-78)

↓ ↓ ↓

信息自由流动 模块化/可组合设计 无信任安全通信

↓ ↓ ↓

自由软件/开源 (1983-91) ← → 密码朋克运动 (1988-92)

↓ ↓

社会共识开发模式 数字货币实验 (1990-2005)

↓ ↓

└──────────── Bitcoin (2008) ──┘

↓

Ethereum (2015)

\`\`\`

\---
<!-- DAILY_CHECKIN_2026-04-07_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->

对密码学有个大概了解
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
