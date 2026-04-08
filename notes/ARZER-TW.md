---
timezone: UTC+8
---

# james_1022

**GitHub ID:** ARZER-TW

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->
2026-04-08

今天把 The Protocol 的設計哲學和演進脈絡整理過了一遍，順便複習 EVM 的執行模型。

設計哲學

以太坊的協議設計圍繞五個原則：Simplicity、Universality、Modularity、Non-discrimination、Agility。

簡單來說就是：底層盡量乾淨，把複雜度往中間層和 L2 推。核心共識層要穩，髒活累活讓編譯器、序列化、客戶端實作去扛。這個分層思路其實跟做系統架構的直覺一致——你不會把業務邏輯塞進 kernel 裡。

Universality 是以太坊跟比特幣拉開距離的關鍵。BTC 的 Script 是刻意受限的，以太坊則是直接給你一台圖靈完備的 VM，愛怎麼寫怎麼寫。代價是狀態管理變得很重，但這就是 Account model 的取捨。

Account model vs UTXO

以太坊選 Account-based 而不是 UTXO，核心理由是要支援有狀態的合約物件。合約不是一次性腳本，它有 storage、有 balance、有 code，長期存在且可被反覆呼叫。UTXO 天然不適合表達這種東西。

Account model 的好處：狀態表示更緊湊、fungibility 更好、合約互動更直觀。壞處：需要 nonce 防重放、狀態膨脹問題更嚴重、並行執行更難搞。後面一堆 EIP 本質上都在幫這個選擇擦屁股。

狀態承諾：MPT

世界狀態透過 Modified Merkle Patricia Trie 組織，區塊頭只存 state\_root，任何節點都能用 root + Merkle proof 驗證某個帳戶或 storage slot 的正確性。輕節點就是靠這套東西活的。

不過 MPT 跑了這麼多年，磁碟 I/O 已經變成瓶頸而不是 hash 計算了，所以才有 Verkle tree 的研究方向——目標是讓 proof 更小，更適合 stateless client。

一句話總結：區塊鏈不是在保存所有狀態，而是在保存對狀態的加密承諾。

EVM 執行模型

整個 EL 的存在就是在跑這個公式：

σ(t+1) = Π(σ(t), B)

σ 是當前狀態，B 是區塊（裡面裝交易），Π 就是 EVM 的狀態轉換函數。Stack-based VM，最大 stack depth 1024，memory 按需擴展但擴展要扣 gas。

交易進來的流程：檢查簽章和 nonce → 預扣最大 gas 費 → 標準化成 Message → 初始化 substate（logs、access list 等）→ 跑 EVM → 成功就提交狀態，失敗就 revert。

所有交易跑完之後還要做最後對帳：算出來的 state\_root、receipts\_root 必須跟區塊頭聲明的完全一致，差一個 bit 整個區塊就是非法的。

Gas 與 EIP-1559

Gas 本質是資源計量單位，同時也是 DoS 防禦機制。EIP-1559 把原本的純競價模式改成 base fee + priority fee：

\- base fee 由協議根據上一個區塊的 gas 使用率動態調整，超過目標就漲（最多 12.5%），低於目標就降，而且 base fee 會被 burn

\- priority fee 是給 proposer 的小費

底層全部用整數運算，完全避開浮點數，確保所有節點算出來的結果一模一樣。

雙客戶端架構

Merge 之後 EL 和 CL 正式分家。EL 負責執行和狀態（跑 EVM、維護 mempool），CL 負責共識（validator 管理、出塊、attestation、finality）。兩者透過 Engine API 在本地溝通。

這個拆分的好處是承認了「執行正確」和「全網同意哪次執行結果」是兩個不同問題，分開之後各自的升級空間大很多。

EELS

EELS（Ethereum Execut
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-06
<!-- DAILY_CHECKIN_2026-04-06_START -->


今天人不在家手邊沒有電腦，先簡單點個名。
<!-- DAILY_CHECKIN_2026-04-06_END -->
<!-- Content_END -->
