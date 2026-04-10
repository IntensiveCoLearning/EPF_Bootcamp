---
timezone: UTC+8
---

# Guikun Yang

**GitHub ID:** 1sh1ro

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->
**EOF 本质上是在给 EVM bytecode 加“文件格式”**：让合约代码从无结构，变成有版本、有分段的容器。

**它最直接的收益是代码和数据分离**，这样客户端、L2 验证器和分析工具更容易区分哪些是可执行代码，哪些只是数据或构造参数。

**EOF 强调部署时校验**：新合约在创建时就检查格式和有效性，减少运行时负担，也为以后加入静态跳转、函数分段、多字节 opcode 等升级打基础。

**EIP-3541 先预留了** `0xEF` **前缀，EIP-3540 再正式定义 EOF v1**，这样可以避免旧合约字节码被误认成 EOF 合约。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->

EVM 是以太坊执行层的核心，它负责在所有节点上以一致的方式执行智能合约代码。它本质上是一个栈式虚拟机，合约最终会被编译成 opcode 来运行，而每一步执行都会消耗 gas，用来限制计算资源并保护网络安全。

学 EVM 不能只把它看成“合约运行器”，还要把它和状态转换联系起来看——交易进入后，EVM 按规则执行字节码，进而读取或修改 memory、storage 和账户状态，所以它其实是以太坊把“输入交易”变成“状态变化”的关键执行引擎。
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->


**Execution Layer 的规范**  
执行层重点放在 **EELS**上，也就是以 Python 写成的执行层参考规范

**EELS 本质上是“可执行的规范”。**  
它是执行客户端核心组件的 Python reference implementation。

**它可以看作 Yellow Paper 的程序员友好版继任者。**  
EELS 是Yellow Paper 的 “spiritual successor”：相比 Yellow Paper 那种偏形式化、偏学术的表达，EELS 更适合开发者直接读代码理解执行语义。

**它的价值不只是“解释规则”，还包括“跟踪升级”。**  
`execution-specs` 仓库不只是放一个静态规范，它还持续跟踪网络升级，并按 fork 给出规范快照。仓库 README 里明确写了它包含 pyspec 和 network upgrades 的规范；渲染后的文档里还能直接看到按 fork 组织的内容，以及 fork 之间的 `diffs`。  
官方博客特别强调，EELS 提供每个 fork 的完整协议快照，还能展示 fork 之间的差异；这比只看 EIP 更容易把握“当前链上真实规则”和“某次升级具体改动”。

**EELS 也和测试直接连着。**  
官方介绍里提到，它可以填充并执行状态测试，并且能结合 `execution-spec-tests` 把测试应用到生产客户端上。所以它不只是“读物”，也是实现验证和原型实验的重要基础。

**执行层“协议规范”和“对外 API 规范”不是一回事。**  
`execution-specs` 管的是执行层协议本身；而 JSON-RPC 这种客户端对外接口，官方放在单独的 `execution-apis` 仓库里。这个仓库把 Ethereum JSON-RPC 描述为所有执行客户端实现的标准接口。
<!-- DAILY_CHECKIN_2026-04-08_END -->
<!-- Content_END -->
