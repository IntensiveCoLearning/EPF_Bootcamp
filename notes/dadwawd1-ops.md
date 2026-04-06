---
timezone: UTC+8
---

# dadwawd1

**GitHub ID:** dadwawd1-ops

**Telegram:** @dadwawd1

## Self-introduction

热爱研究以太坊协议少年，目前正打算全面强化自己，去推销自己

## Notes

<!-- Content_START -->
# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->
以太坊的设计核心理念是简单、普适、模块化。 它起源于 Vitalik 等人在 2013-2014 年的愿景：构建一个图灵完备的虚拟机（EVM），让开发者能直接在区块链上编写任意去中心化应用，而无需底层复杂性。 早期区块链（如比特币）采用 UTXO 模型，而以太坊选择账户模型（Account-based），因为它更灵活、易实现、支持原生合约，且账户是可互换的（fungible）。 数据结构上采用 Merkle-Patricia Trie（MPT）实现高效可验证的状态管理；序列化使用 RLP（后续引入 SSZ 用于 Eth2）；共识从 PoW 演进到 PoS 的 Casper FFG + LMD GHOST（Gasper 组合）；网络层采用 discv5 DHT + GossipSub。 早期（2014-2016）设计强调“未来证明”，通过模块化封装复杂性，为后续 Layer 2 扩展和 Stateless 研究奠定基础.

![fe855952acca1343d848838f5fbf39f8.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/dadwawd1-ops/images/2026-04-06-1775487211309-fe855952acca1343d848838f5fbf39f8.png)

讲解：这张图展现了现在的以太坊框架，用户通过json-RPC发送交易到执行层的mempool通过p2p扩散到全网，共识层的randao（伪随机源）选出proposer从mempool打包生成payload，再通过engine API吧payload交给执行层执行，通过跑EVM更新状态，返回stateRoot，共识层再通过LMD-GHOST + Attestations来确认这个区块，验证者通过beacon APIs投票达成最终目的
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
