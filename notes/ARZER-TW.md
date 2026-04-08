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
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
2026-04-08

今天按第一週的脈絡把 The Protocol 全區過了一遍。

Prehistory（史前歷史）

以太坊不是憑空冒出來的，背後是幾十年密碼學和隱私運動的累積。1980 年代 David Chaum 開始做匿名數位現金研究，1992 年 Eric Hughes、Timothy C. May、John Gilmore 成立了 Cypherpunks，核心信念就是：隱私是開放社會的基本權利，強加密是對抗審查的武器，去中心化系統是最終解方。

比特幣出現之前，密碼朋克們已經做了一堆嘗試：Adam Back 的 Hashcash（PoW 前身）、Wei Dai 的 B-money（匿名分散式電子現金）、Nick Szabo 的 Bit Gold（結合智能合約概念但沒解決雙花）、Hal Finney 的 RPOW（可重用工作量證明，但驗證還是靠中心伺服器）。

2008 年中本聰發表白皮書，2009 年創世區塊上線，第一次真正解決了去中心化雙花問題。但比特幣被設計成極簡計算器——只做貨幣轉帳，腳本功能非常受限。Vitalik 在 2011 年接觸比特幣後意識到需要一個更通用的平台，不只是貨幣，而是能跑任意程式的世界電腦。這就是以太坊的起點。

換個角度講，以太坊的誕生不是偶然，而是去中心化理念被中心化系統壓了太久之後的一次技術反撲。

Architecture（整體架構）

以太坊是分層的雙客戶端架構。三層模型：物理節點（跑客戶端軟體的機器）→ 以太坊網路（全球節點組成的 P2P 網路）→ EVM（裝在節點上的可信計算平台）。

Merge 之後正式分成執行層（EL）和共識層（CL）：

EL 負責交易執行和狀態管理。核心是 EVM，處理智能合約字節碼、維護帳戶餘額和合約狀態、透過 Gas 機制防止濫用。帳戶分兩種：EOA（私鑰控制）和 CA（合約帳戶，由 code 控制）。狀態轉換的本質就是一個函數：σ(t+1) = Π(σ(t), B)，接收區塊裡的交易，跑 EVM，輸出新狀態。主流客戶端有 Geth、Nethermind、Besu、Erigon、Reth。

CL 負責共識、出塊和最終性。驗證者質押 32 ETH 參與出塊和投票，時間被結構化成 slot（12 秒）和 epoch。共識機制用 Casper FFG 處理最終性，LMD-GHOST 處理分叉選擇。

EL 和 CL 透過 Engine API 在本地協作。這個拆分的核心洞察是：「執行正確」和「全網同意哪次執行結果」是兩個不同的問題，分開處理後各自的升級空間大很多。

網路層也是分開的——EL 的 P2P 主要 gossip 交易、維護 mempool，CL 的 P2P 主要 gossip 區塊、推進共識。節點發現用 discv5（基於 Kademlia DHT），資料傳播用 GossipSub。DHT 解決「怎麼找到別人」，Gossip 解決「找到之後怎麼高效傳資料」。

另外值得一提的是 EELS（Ethereum Execution Layer Specification），用 Python 寫的執行層參考規範，算是 Yellow Paper 的工程師友好版。不只能讀，還能直接跑測試、按 fork 看 diff。Repo 在 ethereum/execution-specs，對外 API 則是另一個 repo：execution-apis。

Design Rationale（設計理念）

五個核心原則：Simplicity、Universality、Modularity、Non-discrimination、Agility。

Simplicity：不是說系統小，而是希望任何普通工程師都能理解並實作協議規範，降低對少數核心開發者的依賴。複雜度管理的策略是 sandwich model——底層共識盡量穩定乾淨，把髒活推到中間層（編譯器、序列化、客戶端實作），最外層的 L2 可以接受最多複雜度。

Universality：以太坊跟比特幣最根本的差異。BTC 的 Script 刻意受限，以太坊直接給一台圖靈完備的 VM，讓開發者自己組合任意邏輯。代價是狀態管理變重，但這是通用平台必須承受的。

Modularity：各組件盡量獨立演進，改一塊不會炸掉全局。Dagger、Patricia tree、SSZ、Proto-Danksharding 都是這個思路下的產物。

Non-discrimination：協議不偏好也不打壓特定應用，靠費用機制讓濫用者自己承擔成本，而不是直接禁止某種用途。中立基礎設施的定位。

Agility：協議不是一成不變的，透過 EIP 機制持續演化，接受「協議會變」這件事並且把它制度化。

為什麼選 Account model 而不是 UTXO：以太坊要支援有狀態的合約物件——有 storage、有 balance、有 code，長期存在且可被反覆呼叫。Account model 更適合表達這種世界狀態。好處是狀態更緊湊、fungibility 更好、合約互動更直觀；代價是需要 nonce 防重放、狀態膨脹嚴重、並行執行更難。後面很多 EIP 本質上都在給這個設計選擇補坑。

為什麼用 MPT：區塊鏈節點必須能在不信任對方的前提下驗證狀態。MPT 提供兩個能力：用單一 state\_root 承諾整套狀態，以及為任意狀態項提供 Merkle proof。區塊頭不需要裝整個狀態，只存 root hash，輕節點靠 proof 就能驗證。但 MPT 跑了多年後磁碟 I/O 成了瓶頸，所以有了 Verkle tree 的研究方向——proof 更小、更適合 stateless client。

一句話：區塊鏈不是在保存所有狀態，而是在保存對狀態的加密承諾。

為什麼需要 Gas：資源計量單位，也是 DoS 防禦機制。EIP-1559 把純競價模式改成 base fee + priority fee，base fee 根據上一個區塊的 gas 使用率動態調整並 burn 掉，priority fee 給 proposer 當小費。底層全用整數運算避開浮點數，確保全網算出來的結果一模一樣。

為什麼 EVM 不追求極致性能：首要目標是跨實作一致性。所有客戶端必須對同一筆交易算出完全相同的結果，所以協議更看重規範明確、行為確定、錯誤語義一致，而不是單一實作的速度。EVM 被設計成保守、易驗證、全網可重複執行的抽象機器，犧牲效能換來共識友好性。

Evolution（演進歷史）

Frontier（2015）：最小可用版本，供開發者測試，Gas 上限硬編碼 5000，後來解凍拉到 3,141,592。

Homestead（2016）：從測試進入穩定平台。合約創建 gas 從 21000 提到 53000、修復簽章延展性攻擊、新增 DELEGATECALL。同年 DAO fork 和兩次 DoS 回應升級（Tangerine Whistle / Spurious Dragon）重定價了低估的 opcode。2016 年定義了以太坊的現實主義——鏈上代碼會出錯，參數會被攻擊，社群必須在不中斷系統的前提下做集體選擇。

Byzantium / Constantinople / Istanbul（2017-2020）：為擴展和 PoS 鋪路。降低區塊獎勵、延遲難度炸彈、加入 SNARK/STARK 相關的密碼學 precompile、優化 EVM 成本。2020 年 12 月 Beacon Chain 作為獨立鏈上線，開始 PoS 冷啟動。

London（2021）：EIP-1559 引入，改革費用市場。

The Merge（2022.9.15）：EL 和 CL 合併，PoW 正式退役。不是重啟新鏈，而是保留原有歷史、帳戶、餘額和合約狀態，只把「決定鏈頭的方式」換掉。能耗下降約 99.95%。

Shapella（2023）：啟用質押提款，把 Beacon Chain 的退出和提現能力打通到執行層。

Dencun（2024）：EIP-4844 / Proto-Danksharding，引入 blob 交易，L2 資料發布成本大幅降低。

Pectra（2025）：EIP-7702 增強 EOA 可程式性、EIP-7002 和 EIP-7251 改善驗證者資金控制和資本效率。

整條演進線的邏輯：2015-2020 先活下來 → 2021-2022 重塑費用市場與共識 → 2023-2025 圍繞質押、L2 擴展、帳戶抽象做精細化升級。
<!-- DAILY_CHECKIN_2026-04-09_END -->

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
