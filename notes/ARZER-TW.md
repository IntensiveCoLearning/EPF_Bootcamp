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
# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->
假日打卡
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->

先打個卡 晚點補筆記
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->


\# 2026-04-17

<!-- DAILY\_CHECKIN\_2026-04-17\_START -->

今天讀了 SSZ 和 Merkleization，這兩篇是連在一起的——SSZ 負責把資料序列化成 bytes，Merkleization 負責把序列化後的資料建成 Merkle tree 算出 hash tree root。CL 的狀態驗證就是建立在這兩層之上。

\---

SSZ（Simple Serialize）

SSZ 是 CL 專用的序列化格式，取代了 EL 的 RLP。目前以太坊兩層用不同的序列化：EL 用 RLP，CL 用 SSZ（除了 peer discovery 還在用 RLP）。長期方向是全面遷移到 SSZ。

SSZ vs RLP

RLP 的設計哲學是極簡——只存巢狀 byte array，不定義任何型別。SSZ 則反過來，直接支援具體的資料型別（uint、boolean、vector、list、container 等），而且天生就為 Merkleization 設計。

主要差異：

\- 表達力：RLP 只有 byte string 和 list，其他型別要靠上層協議自己解讀。SSZ 直接支援 uint8/16/32/64/128/256、boolean、固定長度 vector、變長 list、bitvector、bitlist、container。

\- Hashing：SSZ 的結構讓部分資料改變時只需要重新 hash 改變的部分（caching subtree roots），RLP 做不到這種增量更新。

\- Indexing：SSZ 可以不完全反序列化就存取特定欄位（雖然支援程度有限），RLP 完全不支援。

兩者都保證 deterministic serialization——同一份資料永遠序列化成相同的 byte sequence。

Basic Types

uint：直接轉成 little-endian byte array。例如 uint16 的 1025 → 0x0401 → little-endian 變成 01 04。

boolean：True → 0x01，False → 0x00，就一個 byte。

Composite Types

Vector：固定長度的同型別集合，例如 Vector\[uint64, 3\]。序列化就是把每個元素各自轉成 byte array 然後串接起來，不需要額外的長度資訊（因為長度是型別定義的一部分）。

List：變長的同型別集合，有最大長度限制，例如 List\[uint64, 5\]。序列化方式跟 Vector 類似——逐個元素轉換再串接。但因為長度不固定，放在 Container 裡面的時候會多一個 4-byte offset 指向實際資料的起始位置。

這個差異用程式碼看最清楚：同樣放 \[1, 2, 3\]，Vector 直接編碼成 010203，但 List 在 Container 裡會變成 04000000010203（前面多了 4 bytes 的 offset）。

Bitvector / Bitlist：用 bit 存 boolean 值，8 個 bit 塞進 1 個 byte，LSB first。Bitvector 是固定長度，Bitlist 是變長（多一個 sentinel bit 標記結尾）。Bitvector\[10\] 的 1011010010 序列化成兩個 byte：0xB4 0x80。

Container：異質型別的集合，像是 struct。序列化規則是：固定長度的欄位直接放，變長欄位先放一個 4-byte offset 指標，實際資料接在所有固定長度欄位後面。

Fixed-size vs Variable-size 的區分很關鍵：

\- Fixed：uint、boolean、Vector（元素也是 fixed 的話）、Bitvector

\- Variable：List、Bitlist、Vector（元素是 variable 的話）、包含 variable 欄位的 Container

實際例子：IndexedAttestation

IndexedAttestation 是 CL 裡常見的 container，包含三個欄位：

\- attesting\_indices：List\[ValidatorIndex, MAX\_VALIDATORS\_PER\_COMMITTEE\]（變長）

\- data：AttestationData（固定長度的 container）

\- signature：BLSSignature（96 bytes，固定）

序列化時，attesting\_indices 是變長的所以先放 4-byte offset（0xe4000000 = 228），表示它的實際資料從第 228 byte 開始。前面 228 bytes 放的是所有固定長度欄位（slot、index、beacon\_block\_root、source checkpoint、target checkpoint、signature）。

\---

Merkleization

Merkleization 是把 SSZ 序列化後的資料建成 Merkle tree、算出 hash tree root 的過程。CL 靠這個機制讓所有節點能用一個 32-byte hash 來代表整個 Beacon State，快速比對是否一致。

流程

1\. Chunking：把序列化後的資料切成 32-byte 的 chunks，每個 chunk 是 Merkle tree 的一個 leaf

2\. Padding：如果 chunks 數量不是 2 的冪次，補零直到滿足 balanced binary tree 的要求

3\. Tree Construction：兩兩配對 hash，往上層層疊加，直到剩一個 hash——這就是 Merkle root

4\. Mix in Length（List 專用）：List 因為長度可變，需要把長度額外 hash 進去。不然 \[10, 20, 30\] 和 \[10, 20, 30, 0\] 會算出相同的 root，這是安全漏洞

效能上的好處

不需要每次都重新 hash 整棵樹。如果只有部分資料改變，只需要從改變的 leaf 往上重新算到 root，其他 subtree 的 root 可以 cache。對 Beacon State 這種龐大的資料結構來說，這個特性非常關鍵——每個 slot 只有少部分 validator 的狀態會變，不用重算整棵樹。

Generalized Indices

Merkle tree 裡每個節點（包括 leaf 和 internal node）都有一個 generalized index。Root = 1，之後按 2^depth + index 計算。左子節點 = parent _2，右子節點 = parent_ 2 + 1。

這個編號系統讓你可以直接用 index 定位到樹的任何位置，做 Merkle proof 的時候非常方便——只需要提供從 target leaf 到 root 路徑上的 sibling hashes。

Multiproof 的例子：要驗證 index 9 的資料，需要 index 8（sibling）、index 5（uncle）、index 3（更上一層的 uncle）的 hash。把這些 hash 逐層往上算，最後算出的 root 如果跟已知的 root 一致，就證明 index 9 的資料沒被篡改。

這對 light client 極其重要——不需要下載整個 Beacon State，只要拿到目標資料加上一小撮 proof nodes 就能驗證。

Hash Tree Root 的計算

Basic types：直接 pad 到 32 bytes 就是一個 chunk，merkleize 就是它自己。

Composite types（Container）：遞迴計算每個 field 的 hash tree root，把這些 root 當作 leaves 再 merkleize 一次。

實際走一遍 IndexedAttestation 的 merkleization：

1\. attesting\_indices（List）：先序列化 + padding 到 32-byte chunks → merkleize → 然後把 list length 作為第二個 leaf 跟 merkle root 再 hash 一次（mix in length）

2\. data（AttestationData Container）：遞迴處理每個 field（slot、index、beacon\_block\_root 各自 pad 成 32 bytes；source 和 target 是 Checkpoint container 所以再遞迴一層）→ 五個 field root 合起來 merkleize

3\. signature（BLSSignature，96 bytes）：切成三個 32-byte chunks → merkleize

4\. 最後把上面三個 root 當 leaves → merkleize → 得到 IndexedAttestation 的 hash tree root

!\[Merkleization of IndexedAttestation\]([https://epf.wiki/images/merkelization-IndexedAttestation.png](https://epf.wiki/images/merkelization-IndexedAttestation.png))

整個過程是嚴格確定性的——同一份 IndexedAttestation 在任何節點上算出來的 hash tree root 一定相同。這就是共識的數學基礎。

Summaries and Expansions

SSZ 的 merkleization 天然支援 summary/expansion 模式。例如 BeaconBlockHeader 是 BeaconBlock 的 summary——兩者的 hash tree root 相同（因為 header 裡的 body\_root 就是 body 的 merkle root），但 header 只有幾個 field 而 block 有完整的 body。

這讓 proposer slashing 之類的操作可以只用 header 就完成驗證，不需要完整的 block data。

\---

小結

SSZ + Merkleization 是 CL 的資料基礎。SSZ 解決「怎麼把複雜的資料結構變成 bytes」，Merkleization 解決「怎麼用一個 32-byte hash 代表任意大的資料結構，並且支援高效的增量更新和輕量級證明」。

跟 EL 的 RLP + MPT 對比：RLP 只管序列化，MPT 是獨立的資料結構；SSZ 則把序列化和 tree structure 設計成一體的——序列化的結果直接就是 Merkle tree 的 leaves，不需要額外的轉換步驟。這是 SSZ 比 RLP 更適合 CL 需求的根本原因。

<!-- DAILY\_CHECKIN\_2026-04-17\_END -->
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->



\# 2026-04-16

<!-- DAILY\_CHECKIN\_2026-04-16\_START -->

今天開始進入 Consensus Layer 的部分，讀完了 CL Overview 和 CL Specs。從 EL 跳到 CL 視角完全不同——EL 關心的是「交易怎麼執行、狀態怎麼轉換」，CL 關心的是「誰有權出塊、怎麼讓全網同意同一條鏈」。

\---

共識層要解決的根本問題

幾萬個獨立節點分散在全世界，跑在消費級硬體上，透過不可靠的網路連線溝通。節點可能掉線、延遲、設定錯誤，甚至有人故意搞破壞。在這種環境下讓所有節點對同一份帳本達成一致——這就是共識協議要做的事。

學術上叫 Byzantine Fault Tolerance（BFT）：系統在部分節點故障或惡意行為的情況下仍然能正常運作。

一個重要的認知校正：PoW 和 PoS 本身不是共識協議，而是 Sybil resistance mechanism——讓參與協議有成本（算力或質押資本），防止攻擊者用低成本灌入大量假節點。真正的共識協議建立在它們之上，透過 fork choice rule 來決定「跟哪條鏈」。

以太坊的共識協議

以太坊的共識實際上是兩個協議 bolted together：

\- LMD GHOST：fork choice rule，決定「當前鏈頭在哪」

\- Casper FFG：finality gadget，決定「哪些區塊已經不可逆」

兩者結合起來叫 Gasper。

The Merge

2022 年 9 月 15 日，Paris hard fork 用 Terminal Total Difficulty（TTD）而不是 block height 來觸發切換——避免惡意分叉的風險。到達 TTD 的最後一個 PoW 區塊之後，Beacon Chain 正式接管出塊權。

Beacon Chain 從 2020 年 12 月 1 日就開始以獨立鏈的形式運行，Merge 只是把它跟 mainnet 的 EL 接在一起。

Beacon Chain 的結構

時間單位

Slot = 12 秒，是區塊可能被加入鏈上的最小時間單位。每個 slot 有一個被選中的 validator 來 propose block，但 slot 可以是空的（proposer 掉線就沒 block）。

Epoch = 32 slots = 384 秒（6.4 分鐘）。Epoch 是做 checkpoint、justification、finalization 的基本單位。

!\[Slots and Epochs\]([https://epf.wiki/images/cl/slots-and-epochs.png](https://epf.wiki/images/cl/slots-and-epochs.png))

Validator

要當 validator 得先質押至少 32 ETH 到 deposit contract。最大有效餘額 2048 ETH。Validator 的主要工作：

\- Propose block：被 RANDAO 偽隨機選中時，組裝區塊並廣播

\- Attest：對其他人 propose 的 block 投票，表示「我認為這個 block 是有效的」

\- 參與 FFG 投票：在每個 epoch 對 checkpoint 投票

Validator 的選擇引入了密碼學隨機性（RANDAO + VDF），防止可預測和操控。

!\[Validator Selection\]([https://epf.wiki/images/cl/validators.png](https://epf.wiki/images/cl/validators.png))

Committee

Validator 被分組成 committee，每個 slot 至少一個 committee，每個 committee 至少 128 人。攻擊者控制 2/3 committee 的機率不到一兆分之一。

每個 epoch 開始時，RANDAO 把所有 validator 均勻打散到 32 個 slot，每個 slot 再細分成 committee。Validator 在一個 epoch 裡只會被分到一個 slot、一個 committee。

!\[RANDAO\]([https://epf.wiki/images/cl/RANDAO.png](https://epf.wiki/images/cl/RANDAO.png))

Committee 的重要優化：同一個 committee 裡如果多個 validator 做了相同的 LMD GHOST 和 FFG 投票，他們的簽名可以被聚合成一個 aggregate signature，大幅減少資料量。

!\[Committees\]([https://epf.wiki/images/cl/committees.png](https://epf.wiki/images/cl/committees.png))

上圖展示了三個 slot 的情況：Slot 1 正常出塊和 attest；Slot 2 有個 validator miss 了新 block 所以 attest 到前一個；Slot 3 所有 validator 按 LMD GHOST rule attest 到同一個 head。

Attestation

Attestation 是 validator 的投票，權重由 validator 的 stake 決定。每個 attestation 同時包含兩種投票：

\- LMD GHOST vote：投給某個 beacon chain head（哪個 block 是鏈頭）

\- FFG vote：投給某個 checkpoint（source checkpoint + target checkpoint）

每個 validator 每個 epoch 提交一次 attestation，有 32 個 slot 的機會被 include 上鏈。越早被 include 獎勵越高。

Checkpoint 和 Finality

每個 epoch 結束時會產生一個 checkpoint——該 epoch 第一個 slot 的 block（如果那個 slot 是空的，就往前找最近的 block）。

Finality 的流程：

1\. Validator 對 checkpoint 做 FFG 投票（source = 上一個 justified checkpoint，target = 當前 epoch 的 checkpoint）

2\. 當一個 checkpoint 收到 supermajority（2/3 以上的 total staked ETH）的投票，它被 justified

3\. 如果下一個 epoch 的 checkpoint 也被 justified，前一個 justified checkpoint 就被 finalized

4\. Finalized 的 block 及其之前所有 block 都不可逆

!\[Checkpoints\]([https://epf.wiki/images/cl/checkpoints.jpg](https://epf.wiki/images/cl/checkpoints.jpg))

!\[Finalization\]([https://epf.wiki/images/cl/finalization.png](https://epf.wiki/images/cl/finalization.png))

正常情況下 finality 需要 2 個 epoch（約 12.8 分鐘）。考慮到交易平均在 epoch 中間被 include，實際的 transaction finality 大約 14 分鐘。不同的應用場景可以根據需求決定是等完整 finality 還是用更早的 safety threshold。

Supermajority 是按 balance 加權的，不是按人頭。如果有三個 validator 分別質押 8、8、32 ETH，supermajority 需要 32 ETH 那個 validator 的票。

獎懲機制

Attester 獎勵：attestation 跟多數 validator 一致、被 include 在 finalized block 裡，獎勵最高。

Attester 懲罰：沒 attest 或 attest 到沒被 finalize 的 block 會被扣錢。

Proposer 獎勵：成功 propose 被 finalize 的 block 大約能拿到總獎勵的 1/8 加成。

Slashing：嚴重違規行為，至少扣 1/32 的 balance 並被強制退出。四種 slashable offense：

\- Double Proposal：同一個 slot propose 兩個 block

\- LMD GHOST Double Vote：同一個 slot attest 到不同的 head

\- FFG Surround Vote：新的 FFG vote 包圍或被之前的 FFG vote 包圍

\- FFG Double Vote：同一個 epoch 對不同 target 投 FFG vote

Whistleblower 舉報 slashable offense 可以拿獎勵（目前給 block proposer）。

Inactivity Leak：如果超過 4 個 epoch 沒有 finality，不活躍的 validator 會被加速扣錢直到他們的 balance 降到被強制退出，讓剩下的活躍 validator 重新湊到 2/3 majority 恢復 finality。這是極端情況下的自癒機制。

Validator 生命週期

質押 32 ETH 啟動 → 被 Beacon Chain activate → 正常運作（propose + attest）→ 自願退出（需服務至少 2048 epochs ≈ 9 天）或被 slash → 退出後等 4 個 epoch 才能提款（這段期間仍可被 slash）。

正常退出：約 27 小時可提款。被 slash：要等約 36 天（8192 epochs）。

!\[Validator Lifecycle\]([https://epf.wiki/images/cl/validator-lifecycle.png](https://epf.wiki/images/cl/validator-lifecycle.png))

為了防止 validator set 劇烈變動，每個 epoch 能啟動和退出的 validator 數量有上限。

Blobs（EIP-4844）

每個 block 可以帶 3~6 個 blob sidecar。Blob 用 KZG commitment 驗證，需要 Trusted Setup（Deneb 之前做了 KZG Ceremony）。

對節點營運者的影響主要是儲存——blob 預設保留 4096 epochs，大約需要額外 52~104 GB 的儲存空間。過期的 blob 會被 prune。

CL Specification

CL 的規範叫 Pyspec，用 Python 寫的可執行規範，跟 EL 的 EELS 定位類似。Pyspec 同時是客戶端實作的參考和測試向量的來源。

設計基於 Gasper 論文（結合 Casper FFG 和 LMD GHOST 的形式化描述）。

\---

小結

CL 的核心就是在解決「誰出塊、怎麼投票、什麼時候算 final」這三個問題。Slot/Epoch 結構化了時間，RANDAO 提供隨機性，Committee 分散了驗證責任，LMD GHOST 選鏈頭，Casper FFG 做 finality，獎懲機制對齊了 validator 的經濟利益和網路健康。

跟 EL 的關係：CL 決定「在什麼 context 下出塊」（透過 forkChoiceUpdated），EL 負責「在這個 context 下組裝和執行交易」（透過 newPayload / getPayload）。CL 是指揮官，EL 是執行者。

<!-- DAILY\_CHECKIN\_2026-04-16\_END -->
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->




期中考请假
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->





\# 2026-04-13

<!-- DAILY\_CHECKIN\_2026-04-13\_START -->

今天看了 EOF、Precompiled Contracts 和 Block Production 三篇。前兩篇是 EVM 的擴展機制（一個改 bytecode 格式、一個在 EVM 外面跑原生計算），第三篇是區塊從無到有怎麼被組裝出來的。

\---

EOF（EVM Object Format）

現狀

先講結論：EOF 在 2025 年 4 月 28 日被正式從 Fusaka hard fork 移除了。原因包括一次改太多東西（十幾個 opcode 的增刪）、要永久維護 legacy EVM 和 EOF 兩套執行環境的負擔、社群反對意見在多個升級週期裡沒被有效解決、以及會拖慢 PeerDAS 的排程。EOF 的支持者可以在下一個 Glamsterdam fork 重新提案，但目前沒有排入任何計劃。

所以以下內容當作「理解設計意圖」來讀，不是當前生效的規範。

要解決什麼問題

現在的 EVM bytecode 是一串沒有結構的 byte sequence。EVM 要在 runtime 掃描 JUMPDEST、沒辦法在部署時做完整的靜態分析、code 和 data 混在一起分不清楚。

EOF 想做的事：給 bytecode 加上結構化的 container format，讓 EVM 能在 deploy time 做一次性驗證（stack depth、結構完整性、沒有非法 opcode），之後執行時就不用再做這些檢查。

Container 結構

EOF1 的 bytecode 以 0xEF00 magic 開頭，後面跟 version、section headers、0x00 terminator，然後是實際的 section 內容：  
Magic (0xEF00) | Version | (section\_kind, section\_size)+ | 0x00

\[types section\] \[code section 0\] \[code section 1\] ... \[data section\]  
Code 和 data 被明確分開。Types section 定義每個 function 的 input/output signature。

主要改動

控制流：用 RJUMP / RJUMPI / RJUMPV（static relative jump，帶 signed immediate offset）取代 JUMP / JUMPI（dynamic jump）。不再需要 JUMPDEST。這讓控制流在 deploy time 就能完全確定。

Subroutine：支援多個 code section，引入 CALLF / RETF 做 intra-contract function call，JUMPF 做 tail-call optimization。

Stack 存取：SWAPN / DUPN / EXCHANGE 可以存取 stack 上任意位置（最多 256+），不再被 DUP16 / SWAP16 限制。

Data 存取：新的 DATALOAD / DATALOADN / DATASIZE / DATACOPY 取代 CODECOPY，安全地讀取 data section。

外部呼叫：EXTCALL / EXTDELEGATECALL / EXTSTATICCALL 取代舊的 CALL / DELEGATECALL，拿掉 gas 參數改用 63/64 rule。

合約建立：EOFCREATE / RETURNCODE 取代 CREATE / CREATE2。TXCREATE 讓 EOA 直接部署 EOF 合約。

還有 PAY opcode——轉 ETH 但不觸發 recipient 的 code，直接從根本上防 reentrancy。

Legacy vs EOF 對照

| | Legacy | EOF |

|---|---|---|

| 結構 | 無結構的 byte stream | 明確的 section（types, code, data） |

| 驗證 | Runtime，有限 | Deploy-time，完整 |

| 跳轉 | Dynamic JUMP/JUMPI | Static relative RJUMP/RJUMPI |

| Stack 驗證 | Runtime 才知道 overflow/underflow | 靜態分析保證不會 |

| Data | 和 code 混在一起 | 獨立的 data section |

| Code introspection | CODECOPY, EXTCODESIZE | 廢棄，換成更安全的替代品 |

被移除的核心爭議就是：這些改進確實有意義，但一次改這麼多，值不值得承受相應的複雜度和維護成本？反對方認為可以拆成更小的、targeted 的 EIP 逐步達成相同目標。

\---

Precompiled Contracts

Precompile 是什麼

一組特殊帳戶，住在 0x01 到 0x0a 這幾個固定地址上，每個帳戶內建一個原生函數。跟普通合約帳戶不同的是：precompile 由 EL 客戶端直接實作（不是 EVM bytecode），在 EVM 外面跑，所以 EXTCODESIZE 查到的是 0。但呼叫方式跟外部合約呼叫一模一樣（用 CALL / STATICCALL），對開發者來說使用上沒差別。

Precompile 被自動加入交易的 accessed\_addresses（EIP-2929），所以不用額外付 cold access 的 gas。

為什麼不用 opcode

兩個原因：opcode 空間只有 1 byte（256 個），要省著用；precompile 在 EVM 外面用 optimized native library 跑，效率遠高於 EVM 內部執行。

Vitalik 在 Prehistory 裡提過，他們原本有更野心勃勃的 "native contracts" 方案——讓 miner 投票降低某些合約的 gas price，但沒辦法做到 cryptoeconomically safe（攻擊者可以用 trapdoor 讓自己跑得快然後投票壓低 gas price 來 DoS 網路），所以退而求其次選了預定義的 precompile。

目前十個 precompile

| 地址 | 名稱 | 功能 | 引入時間 |

|------|------|------|---------|

| 0x01 | ECRECOVER | 從簽名恢復 public key | Frontier |

| 0x02 | SHA2-256 | SHA-256 hash | Frontier |

| 0x03 | RIPEMD-160 | RIPEMD-160 hash | Frontier |

| 0x04 | IDENTITY | 原封不動返回 input | Frontier |

| 0x05 | MODEXP | 任意精度 modular exponentiation | Byzantium |

| 0x06 | ECADD | 橢圓曲線加法（alt\_bn128） | Byzantium |

| 0x07 | ECMUL | 橢圓曲線標量乘法 | Byzantium |

| 0x08 | ECPAIRING | 橢圓曲線 pairing check | Byzantium |

| 0x09 | BLAKE2 | BLAKE2 壓縮函數 | Istanbul |

| 0x0a | KZG POINT EVAL | 驗證 KZG proof | Cancun |

Gas cost 直接跟 input 綁定——fixed input 就是 fixed cost，有些像 MODEXP 是根據「每秒消耗多少 gas」的 benchmark 來定價的。重點是要防 DoS：gas cost 必須準確反映實際計算量。

底層用 optimized library 執行是效率的來源，但也是風險——library 出 bug 會影響整個協議層。所以 precompile 有大量測試（比如 MODEXP 在 execution-spec-tests 裡有專門的測試集）。Precompile 不做 nested call，也是安全考量。

新增和移除的趨勢

提案中的新 precompile：BLS12-381（EIP-2537）、secp256r1（RIP-7212）、Verkle proof 驗證（EIP-7545）、Poseidon hash（EIP-5988）。

社群對新增 precompile 有阻力——測試面和維護成本高。有個想法是先在 L2 上 prototype，穩定了再上 mainnet。另一個方向是 "progressive precompiles"，用 CREATE2 在確定性地址部署合約版本，等 native precompile 上線後自動切換。

也有人提議移除過時的 precompile（IDENTITY 被 MCOPY 取代了、RIPEMD-160 和 BLAKE2 用得少），但不是直接刪，而是遷移成合約實作，功能不變但 gas 會變貴。

\---

Block Production

整體流程

CL 選出某個 validator 當這個 slot 的 proposer → CL 透過 Engine API 的 forkChoiceUpdated（帶 payload attributes）通知 EL 開始 build block → EL 從 mempool 撈交易、逐筆在 EVM 裡執行、累積狀態變更 → EL 組裝完整區塊 → CL 用 getPayload 取回 payload → CL 包進 beacon block 廣播出去。

Validator 不一定只用自己 EL 產的 block，也可以用外部 builder 的 block（這就是 PBS）。

!\[Payload Building Routine\]([https://epf.wiki/images/el-architecture/payload-building-routine.png](https://epf.wiki/images/el-architecture/payload-building-routine.png))

Build 的簡化邏輯

用 Geth 的 pseudo code 來理解：

\`\`\`go

func build(env environment, pool txpool.Pool, state state.StateDB) (types.Block, state.StateDB, error) {

var gasUsed = 0

var txs \[\]types.Transactions

for ; gasUsed < 30\_000\_000 || !pool.Empty(); {

transaction := pool.Pop() // 從 pool 拿最有價值的交易

res, gas, err := [vm.Run](http://vm.Run)(env, transaction, state)

if err != nil {

continue // 交易無效就跳過，繼續撈下一筆

}

gasUsed += gas

transactions = append(transactions, transaction)

}

return core.Finalize(env, transactions, state) // 算 roots，組裝完整區塊

}

\`\`\`

幾個重點：

env 來自 CL，包含 timestamp、block number、parent block、base fee、withdrawals——CL 決定「在什麼 context 下建這個區塊」。

Transaction pool 假設是按 fee 排好序的，每次 Pop 拿到的是下一筆最有利可圖的交易。目標是 build 出最 profitable 的 block。

Gas limit 是硬上限（mainnet 目前約 30M），塞到滿或 pool 空了就停。

交易執行失敗不會讓整個 build 失敗——跳過那筆繼續做。失敗原因通常是 pool 有點過時（nonce 對不上、balance 不夠之類的）。

Finalize 的時候要算 transactions root、receipts root、withdrawals root，Merkle 化之後塞進 header。

Geth 實作的實際流程

真正的 Geth code 比上面的 pseudo code 複雜不少：

1\. engine\_forkchoiceUpdatedV2 被 CL 呼叫，EL 啟動 block building

2\. buildPayload 先建一個空 block（確保不會 miss slot），然後開一個 goroutine 去填交易

3\. goroutine 裡呼叫 getSealingBlock，把 noTxs 設成 false（要填交易）

4\. 請求透過 getWorkCh channel 送出，被 mainLoop 裡的 listener 接住

5\. generateWork 開始填交易

6\. fillTransactions 從 mempool 撈所有 pending 交易（包括 blob 交易），按 fee 排序

7\. commitTransactions 逐筆檢查是否還有足夠 gas，逐筆 commit

8\. 每筆交易透過 applyTransaction → core.ApplyTransaction 在 EVM 裡執行

9\. 執行成功就更新 state、加入 block；失敗就跳過

10\. 全部交易處理完，CL 用 getPayload 取回填好交易的 payload

11\. CL 把 payload 放進 beacon block 廣播

先建空 block 再 async 填交易這個設計很巧——保證在任何情況下 validator 都有東西可以 propose，不會因為 build 太慢而 miss slot。

\---

小結

EOF 代表了「EVM 應該長什麼樣」的理想願景——結構化 bytecode、deploy-time validation、靜態控制流，但因為改動幅度太大被暫時擱置。Precompile 則是 EVM 能力不足時的務實補丁——把密碼學重活丟到 native code 去跑。Block production 串起了 CL 和 EL 的協作：CL 決定「誰來出塊、在什麼 context 下出」，EL 負責「從 mempool 撈交易、跑 EVM、組裝區塊」。

三篇合在一起看，就是 EVM 的邊界在哪、超出邊界怎麼辦、以及 EVM 產出的結果怎麼被打包成區塊上鏈。

<!-- DAILY\_CHECKIN\_2026-04-13\_END -->
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->








\# 2026-04-11

<!-- DAILY\_CHECKIN\_2026-04-11\_START -->

今天把 EL 的基礎設施層掃完了：Data Structures、RLP、DevP2P、JSON-RPC。前兩篇是「資料怎麼存、怎麼編碼」，後兩篇是「節點之間怎麼找到彼此、外部怎麼跟節點溝通」。

\---

Data Structures：EL 的資料結構

Merkle Tree

Merkle tree 是 hash-based 的二元樹。葉節點放資料，每個非葉節點是子節點的 hash。從底往上算，最終得到一個 Merkle Root。

重點在於防篡改——hash function 的雪崩效應讓任何一個葉節點的微小改動都會導致 root hash 完全不同。這個 root 存在 block header 裡，所以不需要下載整棵樹就能驗證資料完整性。

!\[Merkle Tree\]([https://epf.wiki/images/merkle-tree.jpg](https://epf.wiki/images/merkle-tree.jpg))

Patricia Trie

Patricia Trie（Radix tree）跟 Merkle tree 的目的不一樣——Merkle 是為了驗證，Patricia 是為了高效存取。

特點是邊可以標記字元序列（不只是單一字元），而且只有一個子節點的 node 會跟 parent 合併。這兩個壓縮手段讓 trie 比普通的前綴樹省空間很多。值只存在葉節點。

!\[Patricia Trie\]([https://epf.wiki/images/data-structures/patricia-trie.png](https://epf.wiki/images/data-structures/patricia-trie.png))

Merkle Patricia Trie（MPT）

以太坊把兩者結合：從 Merkle tree 拿到密碼學驗證能力，從 Patricia trie 拿到高效的 key-value 存取。這就是 MPT，以太坊 EL 的核心資料結構。

三種節點：

\- Branch Node：17 元素的 array（16 個分支 + 1 個 value），是 trie 裡的分岔路口

\- Extension Node：壓縮節點，當 branch 只有一個 child 時把路徑壓起來

\- Leaf Node：存實際的 key-value 資料

key 的最小單位是 nibble（半個 byte，一個 hex digit），所以每個 branch 最多 16 個分支（0-f）。

!\[MPT Diagram\]([https://epf.wiki/images/data-structures/merkle-patricia-trie.png](https://epf.wiki/images/data-structures/merkle-patricia-trie.png))

以太坊裡的四棵 Trie

!\[Four Tries\]([https://epf.wiki/images/tries.png](https://epf.wiki/images/tries.png))

每個區塊都有自己的 Transaction Trie、Receipt Trie 和 State Trie，root hash 存在 block header。每個合約還有一棵 Storage Trie，root hash 存在對應帳戶的 storageRoot 欄位。

Transaction Trie：key = RLP(交易 index)，value = RLP(交易)。區塊確認後就不會再變。

Receipt Trie：結構跟 Transaction Trie 類似，key 也是交易 index，value 是對應的 receipt。用來驗證交易確實被執行過而不需要重跑一次。Snap sync 的時候下載 block body 會包含交易和 receipt，本地重建 receipt trie 後跟 header 裡的 receiptsRoot 比對。

!\[Receipt Trie\]([https://epf.wiki/images/data-structures/receipt-trie.png](https://epf.wiki/images/data-structures/receipt-trie.png))

World State Trie：key = keccak256(20-byte address)，value = RLP(\[nonce, balance, storageRoot, codeHash\])。這棵 trie 是活的——每個區塊執行完交易後都會更新，不像 transaction/receipt trie 是每個區塊重建的。Full node 只保留當前狀態和足夠做 reorg 的歷史節點，archive node 才保留所有歷史狀態。

!\[World State Trie\]([https://epf.wiki/images/eth-tries.png](https://epf.wiki/images/eth-tries.png))

用 keccak256 hash 地址當 key（而不是直接用地址）是刻意的——防止攻擊者構造特定 key 讓 trie 變得極度不平衡，導致查詢時間暴增（DoS）。

Storage Trie：每個合約帳戶有一棵，key = keccak256(storage slot index)，value = RLP(slot value)。SSTORE 寫、SLOAD 讀。EOA 的 storage trie 永遠是空的。Storage trie 不嵌在 world state trie 裡面，而是透過 storageRoot 引用。

Verkle Tree

以太坊路線圖裡 The Verge 的核心。目標是取代 MPT，讓 stateless client 變得可行。

MPT 的問題：二元/16 叉樹深度大，witness data（驗證某個葉節點需要的所有 sibling hash）非常大。1000 個葉節點的 binary Merkle tree 需要約 4MB 的 witness。

Verkle tree 的做法：用寬度換深度（目前提案每個節點 256 個 children），中間節點用 vector commitment 代替普通 hash。Vector commitment 可以被聚合成一個 compact proof，witness 從 4MB 降到約 150KB。

!\[Verkle Tree\]([https://epf.wiki/images/verkle\_tree\_structure.png](https://epf.wiki/images/verkle_tree_structure.png))

key 也從 20 bytes 改成 32 bytes，其中 31 bytes 當 stem、最後 1 byte 給 storage。stem 相同的資料會共享同一個 commitment（開同一個 commitment 比較便宜），這個設計讓鄰近的 code chunks 或 storage slots 可以被高效批量驗證。

轉換到 verkle tree 是個大工程——要從現有 MPT 生成 verkle 資料，計算量和空間都很大，目前還在研究分發和驗證的方案。

\---

RLP：序列化

RLP（Recursive Length Prefix）是 EL 的序列化協議。以太坊用兩套序列化格式：EL 用 RLP，CL 用 SSZ。

設計哲學很極端——RLP 只做一件事：存巢狀的 byte array 結構。不定義任何資料型別（boolean、integer、float 都沒有），留給上層協議自己解讀。追求的是 byte-perfect 一致性，確保不同語言的實作編碼出完全相同的結果。

編碼規則其實不複雜，就是看第一個 byte 落在哪個範圍：

\- 單一 byte 且 < 0x80 → 直接就是自己

\- 0x80 → null / 空字串 / false

\- 0x81-0xB7 → 短字串（長度 ≤ 55），prefix = 0x80 + length

\- 0xB8-0xBF → 長字串（長度 > 55），prefix 後面先放「長度的長度」

\- 0xC0 → 空 list

\- 0xC0-0xF7 → 短 list（payload ≤ 55 bytes），prefix = 0xC0 + total length

\- 0xF8-0xFF → 長 list（payload > 55 bytes）

例子：\["cat", "dog"\] 編碼過程：

1\. "cat" = 0x83 0x63 0x61 0x74（0x83 = 0x80 + 3）

2\. "dog" = 0x83 0x64 0x6f 0x67

3\. 總 payload = 8 bytes

4\. list prefix = 0xC8（0xC0 + 8）

5\. 結果：0xC8 0x83 0x63 0x61 0x74 0x83 0x64 0x6f 0x67

解碼就是反過來——看第一個 byte 判斷類型，算出長度，遞迴解析。

\---

DevP2P：EL 的網路層

DevP2P 分兩層：Discovery（找到其他節點）和 Transport（安全通訊）。Discovery 跑 UDP（不需要可靠連線，只要告訴別人「我在這」），Transport 跑 TCP（需要可靠、有序的資料傳輸）。

!\[TCP vs UDP\]([https://epf.wiki/images/el-architecture/tcpudp.png](https://epf.wiki/images/el-architecture/tcpudp.png))

Discovery：Discv4 / Discv5

節點發現的起點是寫死在 code 裡的 bootnodes。新節點先跟 bootnode 玩 PING-PONG——送 PING 過去，收到 PONG 就算 bond 成功。然後送 FINDNODE 請求，bootnode 回一組 neighbours（最多 16 個），再跟這些 neighbours 逐一 bond。

!\[Peer Discovery\]([https://epf.wiki/images/el-architecture/peer-discovery.png](https://epf.wiki/images/el-architecture/peer-discovery.png))

底層用 Kademlia DHT——用 XOR distance 衡量節點之間的「距離」，維護 256 個 k-bucket（每個 bucket 最多 16 個 entry），按 last-seen 排序，12 小時沒回應就踢掉。Lookup 是遞迴的：從最近的已知節點開始問，問到的新節點再繼續問，直到找不到更近的為止。

Discv4 vs Discv5 的主要差異：

| | Discv4 | Discv5 |

|---|---|---|

| 安全性 | 明文 | AES-GCM 加密 |

| Handshake | 無 | WHOAREYOU challenge-response |

| 服務發現 | 沒有 | Topic-based lookup |

| 擴展性 | 靜態 | 支援多種 identity scheme |

大部分主流 EL 客戶端（Geth、Nethermind、Reth）已經支援 Discv5，Besu、Erigon 還在過渡中。

ENR（Ethereum Node Record）

節點的連線資訊用 ENR 格式表示（EIP-778），包含 IP、TCP/UDP port、public key、sequence number，用 RLP 編碼，最大 300 bytes，有密碼學簽名保證真實性。

除了 ENR 還有兩種舊格式：multiaddr 和 enode（URL-like），但 ENR 是現在的標準。

RLPx：傳輸層

找到節點之後，RLPx 負責建立安全連線和傳輸資料。

!\[RLPx Communication\]([https://epf.wiki/images/el-architecture/rlpx-communication.png](https://epf.wiki/images/el-architecture/rlpx-communication.png))

連線建立流程：

1\. Initiator 生成 secp256k1 ephemeral key pair

2\. 送 auth message（含 ephemeral pubkey + nonce）給 recipient

3\. Recipient 驗證後送 ack 回來

4\. 雙方用 ECDH 算出 shared secret，derive 出 AES key 和 MAC key

5\. 送 Hello message 交換 port、node ID、supported protocols

6\. 開始加密通訊

用 ECIES 做 handshake，AES-128-CTR 加密，HMAC-SHA-256 做完整性驗證。Ephemeral key 每次 session 都重新生成，session 結束就丟掉——這提供了 forward secrecy，就算長期私鑰被洩漏，過去的通訊紀錄也解不開。

Frame 結構：header-ciphertext | header-mac | frame-ciphertext | frame-mac。支援 multiplexing，同一條 RLPx 連線可以跑多個 subprotocol。

應用層 subprotocol：

\- eth：區塊鏈資料交換（block propagation、tx relay）

\- snap：狀態同步（下載 state trie 片段）

\- les：輕客戶端支援

\- portal：去中心化的 state/block/tx 檢索網路

\---

JSON-RPC：對外 API

JSON-RPC 是 EL 跟外部世界溝通的介面。錢包查餘額、DApp 送交易、CL 跟 EL 對話（Engine API），底層都是 JSON-RPC。

請求格式固定：

\`\`\`json

{

"id": 1,

"jsonrpc": "2.0",

"method": "eth\_blockNumber",

"params": \[\]

}

\`\`\`

方法按 namespace 分類：eth（跟鏈互動）、debug（查 raw data）、admin（節點設定，敏感）、txpool（交易池）、net（網路資訊）等。不同客戶端可能有自己獨有的 namespace。

常用 eth 方法：

\- eth\_blockNumber：最新區塊號

\- eth\_getBalance：查餘額

\- eth\_call：模擬執行（不上鏈）

\- eth\_estimateGas：估算 gas

\- eth\_getCode：查合約 bytecode

\- eth\_getLogs：查事件日誌

\- eth\_getStorageAt：查 storage slot

Engine API 跟一般 JSON-RPC 不同——跑在獨立的 authenticated endpoint 上，用 JWT 驗證（確認 caller 是 CL），不對外公開。主要方法就是之前讀過的 newPayload 和 forkChoiceUpdated。

傳輸協議不限定：

\- HTTP：request-response，回完就斷

\- WebSocket：雙向，保持連線，適合做 event subscription

\- IPC：同機器上的 process 間通訊，最快但不能遠端

編碼慣例：所有數值和資料都用 0x 前綴的 hex 表示。0x41 = 65，不允許前導零（0x0400 不合法，要寫 0x400）。

\---

小結

這四篇涵蓋了 EL 的「基礎建設」：MPT 定義了資料怎麼組織和驗證、RLP 定義了資料怎麼序列化成 bytes、DevP2P 定義了節點之間怎麼發現彼此和安全通訊、JSON-RPC 定義了外部怎麼跟節點互動。

串起來的流程就是：使用者透過 JSON-RPC 送交易 → EL 客戶端驗證後放進 mempool → 透過 DevP2P gossip 給其他節點 → 交易被打包進區塊 → 用 RLP 編碼 → 存進 MPT 結構 → root hash 寫入 block header。

<!-- DAILY\_CHECKIN\_2026-04-11\_END -->
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->









今天有點忙，暫時水個卡。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->










2026-04-10

今天讀完 [epf.wiki](http://epf.wiki) 的 EVM 和 Transaction 兩篇，把 EVM 的內部運作機制和交易從簽署到上鏈的完整生命週期搞清楚了。

* * *

EVM：以太坊虛擬機

以太坊狀態機

以太坊整體可以看成一個 transaction-based state machine。每筆交易是 input，驅動狀態從 S 轉移到 S'。EVM 就是這台狀態機的狀態轉換函數：

Υ(S, I) ⟹ S'

World State 本質上是一個 20-byte address → account state 的映射。

![Ethereum World State](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/ethereum-world-state.jpg)

帳戶分兩種：

-   External Account（EOA）：私鑰控制，沒有 code
    
-   Contract Account：由非空的 EVM code 控制，就是所謂的智能合約
    

為什麼要用虛擬機

不同硬體（x86、ARM、RISC-V）有不同指令集，同一段程式在不同平台上跑可能得到不同結果。傳統做法是針對每個平台編譯成 native binary。虛擬機的解法是加一層抽象：source code 先編譯成 bytecode，再由平台特定的 VM 翻譯成 native code 執行。

![Virtual Machine Paradigm](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/virtual-machine-paradigm.jpg)

好處：可移植性（bytecode 到處跑）和抽象（開發者不用管硬體差異）。JVM 和 LuaVM 都是這個思路。EVM 也一樣——確保全網所有節點對同一筆交易算出完全相同的結果。

EVM 結構

![EVM Anatomy](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/evm-anatomy.jpg)

EVM 的 word size 是 32 bytes（256 bits）。上圖是簡化版，實際還有 Message Frames 和 Transient Storage。EVM 在執行過程中會操作帳戶的 storage、code 和 balance，而且可能涉及多個帳戶的交互。

EVM Bytecode

Bytecode 就是一串 bytes，每個 byte 要嘛是一個 opcode（指令），要嘛是 opcode 的 operand（運算元）。目前只有 PUSH 系列有 operand，PUSHX 代表接下來 X 個 bytes 是 data。

![EVM Assembly](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/opcode-assembly.jpg)

常用 opcode：

| Opcode | 名稱 | 功能 |
| --- | --- | --- |
| 60 | PUSH1 | 把 1 byte 推上 stack |
| 01 | ADD | 加法 |
| 02 | MUL | 乘法 |
| 52 | MSTORE | 寫 word 到 memory |
| 54 | SLOAD | 從 storage 讀 word |
| 55 | SSTORE | 寫 word 到 storage |
| 56 | JUMP | 修改 program counter |
| 5B | JUMPDEST | 標記合法的跳轉目標 |
| f3 | RETURN | 停止執行，回傳 output |
| 35 | CALLDATALOAD | 從 calldata 讀 32 bytes 到 stack |

四個資料儲存位置

Stack

LIFO 結構，push 和 pop 兩個操作。最大深度 1024 items，每個 item 32 bytes，只有 top 16 個 item 可存取（因為 DUP 和 SWAP 最多到 16）。每次合約執行完就清空。

Stack 是 EVM 的 scratchpad：opcode 從 stack top 取值、計算完再 push 回去。

![Stack Addition](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/stack-addition.gif)

上面是最簡單的加法程式：PUSH1 06、PUSH1 07、ADD → 結果 0x0d（十進位 13）。

Program Counter

PC 追蹤下一個要執行的 byte 在 bytecode array 裡的位置（offset）。一般情況下 PC 每執行一個 opcode 就 +1，遇到 PUSH 就跳過 operand bytes，遇到 JUMP 就直接設成 stack top 的值。

![Program Counter](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/program-counter.gif)

JUMP + JUMPDEST 讓 EVM 能做動態控制流——if/else、for loop、internal function call 在底層都是靠 jump 實現的，這也是 EVM 圖靈完備的基礎。

![Jump Opcode](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/jump-opcode.gif)

上面的程式會無限循環地加 7。但無限迴圈在 EVM 上是致命的——會吃光資源、造成 DoS。

Gas

Gas 機制就是為了解決這個問題。Gas 是計算資源的貨幣，每個 opcode 有對應的 gas cost，交易執行時持續扣 gas，扣完就強制停止。這讓 EVM 成為 quasi Turing complete——理論上圖靈完備，但實際上受 gas 限制。

![EVM Gas](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/evm-gas.gif)

Memory

Byte array，理論大小 2^256（實際上無限）。所有位置初始值為零。Memory 補充了 stack 的不足——stack 只能存 32-byte word 且深度有限，memory 允許任意大小的 indexed access。

![EVM Memory](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/evm-memory.gif)

MSTORE 從 stack 取兩個值：offset 和 32-byte value，寫到 memory 對應位置。MSTORE8 只寫 1 byte。MLOAD 讀一整個 word 回 stack。

Memory 是按需擴展的，以 1 word（32 bytes）為單位，擴展要額外付 gas。如果寫入位置超出當前 memory 範圍就會觸發 expansion。

Memory 在交易結束後就消失，不會持久化。

Calldata

交易帶進來的唯讀輸入資料。用 CALLDATALOAD 從指定 offset 讀 32 bytes 到 stack，或用 CALLDATACOPY 複製到 memory。

Storage

Word-addressed word array，2^256 slots，每個 slot 32 bytes。跟 memory 最大的差異：storage 是持久化的，屬於帳戶的一部分，寫入世界狀態。

![EVM Storage](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/evm-storage.jpg)

SSTORE 從 stack 取 slot 和 value 寫入。SLOAD 取 slot 讀回 stack。因為 storage 是全網共識的一部分，讀寫都很貴。

![SSTORE](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/sstore.gif)

![SLOAD](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/sload.gif)

Storage 不是 EVM 本身的一部分，而是 contract account 的。帳戶裡有個 storage root 指向一棵獨立的 MPT。SSTORE 把非零值設回零時可以拿到 gas refund（不是立刻退，而是記到 refund counter，交易結束時抵扣）。

EVM 升級

EVM 的改動通常很保守——大改會破壞既有合約和語言，需要維護多版本 EVM，複雜度爆炸。現有的升級多是加新 opcode 而不破壞舊的，例如：

-   EIP-1153：TSTORE / TLOAD（transient storage）
    
-   EIP-5656：MCOPY
    
-   EIP-6780：弱化 SELFDESTRUCT（不破壞相容性的情況下）
    

比較大的變動是 EOF（EVM Object Format），給 bytecode 定義結構化格式，讓 EVM 更容易解析和驗證。

* * *

Transaction：交易

交易是由 EOA 發出的、經過密碼學簽名的指令。透過 JSON-RPC 提交給 EL 客戶端，再經 DevP2P 廣播到全網。

交易欄位

-   nonce (T\_n)：sender 發出的交易計數。防重放攻擊（Bob 不能重發 Alice 的舊交易）、決定合約地址、允許用同 nonce 替換 pending 交易（加速或取消）
    
-   gasPrice (T\_p)：每單位 gas 付多少 wei（legacy 交易用）
    
-   gasLimit (T\_g)：這筆交易最多用多少 gas
    
-   to (T\_t)：接收者地址，決定交易模式：
    
    -   空 → contract creation
        
    -   EOA 地址 → value transfer
        
    -   Contract 地址 → contract execution
        
-   value (T\_v)：轉多少 wei
    
-   data (T\_d) 或 init (T\_i)：給 EVM 的輸入。creation mode 下是 init bytecode，否則是 input data
    
-   signature (T\_v, T\_r, T\_s)：ECDSA 簽名
    

Contract Creation 實戰

用一個具體例子走一遍。要部署的 runtime code：

```
PUSH1 06  // push 6
PUSH1 07  // push 7
MUL       // 6 * 7 = 42
PUSH1 00  // storage slot 0
SSTORE    // 存到 slot 0
```

Bytecode: `6006600702600055`

但交易的 data 欄位不是直接放 runtime code，而是 init code：

```
<init bytecode> <runtime bytecode>
```

init code 只在建立帳戶時跑一次，它的 return value 就是 runtime code，會永久存到 contract account 裡。之後每次收到交易都跑 runtime code。

Init code 做兩件事：把 runtime code 複製到 memory → RETURN 回去：

```
PUSH1 08   // runtime code 長度
PUSH1 0c   // runtime code 在 init 裡的 offset
PUSH1 00   // memory destination
CODECOPY   // 複製到 memory
PUSH1 08   // return 長度
PUSH1 00   // return 起始位置
RETURN     // 回傳 runtime code
```

完整 init bytecode: `6008600c60003960086000f36006600702600055`

![Contract Creation](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/create-contract.gif)

Contract Execution 實戰

部署完之後，發另一筆交易去執行合約。to 指向合約地址，data 留空（這個合約不需要 input）。執行完後用 cast storage 查 slot 0，拿到 0x2a（十進位 42）。

![Contract Execution](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/ARZER-TW/images/contract-execution.gif)

Receipts

每筆交易執行完（不管成功或失敗）都會產生一個 receipt，包含：

-   Transaction Type
    
-   Status：0（失敗）或 1（成功）
    
-   Gas Used：到目前為止區塊內累計的 gas 消耗
    
-   Logs：事件日誌，包含 logger address、indexed topics（最多 4 個 32-byte）、non-indexed data
    
-   Logs Bloom：256-byte bloom filter，用來快速搜尋相關 log
    

Receipt 會被提交到區塊的 Receipt Trie。

Typed Transactions（EIP-2718）

EIP-2718 統一了交易和 receipt 的格式：Transaction Type（1 byte）+ Payload。判斷方式：

-   第一個 byte ∈ \[0x00, 0x7f\] → typed transaction
    
-   第一個 byte ≥ 0xc0 → legacy transaction（RLP list 編碼的特徵）
    

五種交易類型

Type 0 - Legacy：EIP-2718 之前唯一的格式。現在仍然相容，會被自動轉成 EIP-1559 格式（max\_fee = max\_priority\_fee = gasPrice）。

Type 1 - Access List（EIP-2930）：多了 access\_list 欄位，預先宣告要存取哪些 address 和 storage slot。背景是 2016 年 Shanghai DoS 攻擊——攻擊者發送存取大量地址和 storage 的交易，逼客戶端做大量磁碟 I/O。EIP-2929 提高了 cold access 的 gas cost，EIP-2930 則讓交易可以預載資料來降低成本（warm vs cold access 的區別）。

Type 2 - Dynamic Fee（EIP-1559）：引入 base fee + priority fee。base fee 根據區塊使用率動態調整，目標維持 50% 使用率。base fee 被 burn（減少 ETH 供給、防止 validator 用免費交易操控 gas price），priority fee 給 validator。解決了舊模型下 gas price 劇烈波動、交易卡在 mempool 的 UX 問題。

Type 3 - Blob（EIP-4844）：攜帶 blob 資料的交易。Blob 不存在鏈上，而是像 sidecar 一樣跟區塊一起傳播，header 只存 KZG commitment。資料暫存約 18 天（4096 epochs）。Blob 有自己的 gas price market，獨立於普通 gas。存在的意義是為 L2 提供 data availability——optimistic rollup 需要原始交易資料來生成 fraud proof，blob 確保這些資料在一段時間內可取得。

Type 4 - Set Code（EIP-7702）：讓 EOA 可以指向一個合約地址的 code 來執行。code\_hash 設為 0xef0100 || delegated\_address，0xef 是 EIP-3541 保留的 byte 所以不會跟既有合約衝突。EOA 不是真的持有 code，只是一個指標，可以用另一筆 Type 4 交易移除或更換。解決了純 EOA 的 UX 問題（比如 ERC-20 的 approve/transferFrom 兩步驟），讓 EOA 也能享受帳戶抽象的好處（transaction batching、gas sponsorship 等）。

* * *

小結

EVM 這篇把虛擬機從概念到實作講得很清楚：state machine → virtual machine paradigm → bytecode/opcode → stack/memory/storage/calldata 四個資料位置 → gas 機制。Transaction 這篇則從交易欄位定義出發，用 Foundry 實際走了一遍 contract creation 和 execution，最後梳理了五種交易類型的演進脈絡和各自的設計動機。

兩篇串起來就是：交易是 EVM 的 input，EVM 是處理交易的 engine，處理完的結果（state change + receipt）寫回世界狀態。
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->











2026-04-09

今天深入讀了 [epf.wiki](http://epf.wiki) 上 Execution Layer 的兩篇核心文件：el-specs（EL 規範）和 el-architecture（客戶端架構），把 EL 從規範定義到工程實作的脈絡串起來。

\---

EL Specification：執行層規範

執行層最早由 Yellow Paper 定義，現在最權威的規範是 EELS（ethereum/execution-specs），用 Python 寫的可執行規範。Yellow Paper 的 Paris 版本已經過時，沒涵蓋 Merge 之後的更新。

狀態轉換函數（STF）

整個 EL 的存在就是為了回答兩個問題：這個區塊能不能接在鏈尾？接上之後狀態怎麼變？

公式：σ(t+1) ≡ Π(σ(t), B)

\- σ(t) = 當前區塊鏈狀態

\- B = 當前區塊

\- Π = 區塊層級的狀態轉換函數

\- σ(t+1) = 新狀態

!\[STF Overview\]([https://epf.wiki/images/el-specs/stf\_eels.png](https://epf.wiki/images/el-specs/stf_eels.png))

重點是 σ 不等於程式碼裡的 State class。狀態不是存在某個特定位置的靜態檔案，而是透過 World State Collapse Function 動態計算出來的 Merkle root。這個概念上的區分很重要——數學模型裡的「狀態」和軟體實作裡的「狀態物件」是兩回事。

!\[State Diagram\]([https://epf.wiki/images/el-specs/state.png](https://epf.wiki/images/el-specs/state.png))

STF 的完整流程：

1\. 取得 parent block header

2\. 驗證 excess blob gas（和 parent 比對）

3\. 驗證當前區塊 header（和 parent 比對）

4\. 確認 ommers 欄位為空（Merge 之後不再有 uncle blocks）

5\. 執行區塊內所有交易，產出：gas used、trie roots、logs bloom、新 state

6\. 驗證執行結果和 header 裡宣告的參數一致（state\_root 必須吻合）

7\. 通過就把區塊加到鏈上

8\. 清理超過 255 個區塊的舊資料

9\. 任何驗證失敗就拋 Invalid Block

區塊頭驗證與經濟模型

區塊頭驗證就是一連串數學不等式，確保每個區塊符合協議規則：

\- gas used 不能超過 gas limit

\- gas limit 的變化幅度不能超過 parent 的 1/1024（漸進調整，不能暴力拉）

\- gas limit 最低 5000

\- timestamp 必須嚴格遞增

\- base fee 必須符合 EIP-1559 計算規則

\- parent hash 必須是 parent header 的 KEC(RLP) hash

\- Merge 之後 difficulty = 0, nonce = 0x0000000000000000

\- blob gas 相關欄位的一致性驗證（EIP-4844）

EIP-1559 base fee 動態調整

核心概念：gas target = gas limit / 2（彈性係數 ρ = 2）

\- parent gas used = target → base fee 不變

\- parent gas used > target → base fee 上漲（最多 ~12.5%）

\- parent gas used < target → base fee 下降（最多 ~10%）

\- base fee max change denominator ξ = 8，控制調整速率

!\[Gas Used vs Base Fee\]([https://epf.wiki/images/el-specs/gasused-basefee.png](https://epf.wiki/images/el-specs/gasused-basefee.png))

這張圖可以看到：函數呈現階梯狀線性遞增，最大變異在中點（gas target）處。精確命中 target 時 base fee 上漲約 1%。

底層全部用自然數運算（ℕ），完全避開浮點數。這不是偶然——離散的自然數保證了全網在任何硬體上算出完全一致的結果，不會因為浮點精度差異導致分叉。

長期模擬下（持續滿載 100K 個區塊），base fee 在 200 個區塊內就能從 wei 級別衝到 1 ETH，2000 個區塊內逼近理論上限。但 gas limit 本身是無上界的，會持續增長來適應需求，最終和 base fee 達到新均衡。

!\[Long-term Gas Simulation\]([https://epf.wiki/images/el-specs/gas-limit-max.png](https://epf.wiki/images/el-specs/gas-limit-max.png))

改變 ξ 和 ρ 對經濟模型的影響：

!\[xi parameter effect\]([https://epf.wiki/images/el-specs/xi.png](https://epf.wiki/images/el-specs/xi.png))

!\[rho and xi combined\]([https://epf.wiki/images/el-specs/rho-xi.png](https://epf.wiki/images/el-specs/rho-xi.png))

ξ 越大，fee 調整越平滑；ρ 影響 inflection point 的位置。這兩個參數在 fork 之間不會變，但未來協議升級可以重新指定。

Blob Gas 動態（EIP-4844）

blob gas price 用類指數函數（Taylor expansion 近似）計算：

\- gas used 低於 target（~393K，約 3 blobs/block）時，price 維持 1

\- 超過 target 不會立刻漲價，但 excess gas 開始累積

\- 持續超標導致 excess 突破閾值後，price 指數上升

\- 只要一個區塊的 gas used 低於 target，累積的 excess 就可以被清掉

!\[Blob Gas and Price\]([https://epf.wiki/images/el-specs/blob-gas-and-price.png](https://epf.wiki/images/el-specs/blob-gas-and-price.png))

!\[Normalized Blob Gas\]([https://epf.wiki/images/el-specs/blob-gas-and-price-norm.png](https://epf.wiki/images/el-specs/blob-gas-and-price-norm.png))

區塊執行流程（apply\_body）

Header 驗證通過後進入執行階段。先做輕量驗證可以在不跑 EVM 的情況下就擋掉非法 payload，省下大量計算。

1\. blobGasUsed 初始化為 0

2\. gasAvailable 設為 header 的 gasLimit

3\. 初始化 receipt trie、withdrawal trie、block logs

4\. 存取 Beacon Block Roots Contract（EIP-4788）——讓 EVM 能以 trust-minimized 方式存取 CL 資料

5\. 建構 System Transaction Message（system address → BEACON\_ROOTS\_ADDRESS）

6\. 設定 VM 環境並處理 system call

7\. 清理空帳戶

8\. 逐筆處理交易：decode → 加入 tx trie → 恢復 sender address → 驗證 intrinsic validity → 算 effective gas price → 初始化環境 → 跑 EVM

9\. 處理 validator withdrawals（EIP-4895）

Gas 計算

Intrinsic Gas = 基礎的 21000 + calldata 逐 byte 計費（非零 16 gas、零 4 gas）+ 合約創建額外 32000 + access list 費用

Effective Gas Price：

\- Type 0/1：直接用 gasPrice

\- Type 2/3：baseFee + min(maxPriorityFeePerGas, maxFeePerGas - baseFee)

Upfront Cost = effectiveGasFee + blobGasFee，在執行前先從 sender 扣除。

交易執行六個階段

Stage 1 - Checkpoint σ₀：驗證 → 扣 intrinsic gas → nonce+1 → 扣 upfront cost，這些改動不可逆

Stage 2 - 正規化 & Substate 初始化：把各種交易類型統一成 Message 格式（caller, target, gas, value, data, code, depth...），根據 T\_to 決定是 contract creation 還是 call。初始化 substate（self-destruct set、log series、touched accounts、refund balance、access lists）

Stage 3 - 主要執行 Ξ：這是 EVM 真正跑起來的地方

Machine State μ = (gasAvailable, programCounter, memoryContents, activeWordsInMemory, stackContents)

Stack: 256-bit word，最大深度 1024

Memory: word-addressed byte array，按需擴展（擴展要扣 gas）

PC: 追蹤當前執行到哪個 byte

Single Execution Cycle O：讀取 currentOperation → 操作 stack → 扣 gas → 更新 PC

PC 更新邏輯：

\- JUMP → 跳到 stack top 指定的位置

\- JUMPI → stack top ≠ 0 才跳

\- PUSH1-PUSH32 → PC 跳過 data bytes

\- 其他 → PC + 1

遞迴執行函數 X 的邏輯：

\- 如果觸發 exceptional halting（OOG 等）→ 返回空狀態

\- 如果 currentOp = REVERT → 返回空狀態但保留 output

\- 如果有 output（比如 CALL 系列指令的子 EVM 回傳結果）→ 把 output 寫回 parent memory

\- 否則繼續遞迴呼叫

Normal Halting：RETURN/REVERT 回傳 memory slice，STOP/SELFDESTRUCT 回傳空 tuple

Stage 4/5/6（Provisional → Pre-Final → Final State）：規範裡標註 TODO，尚未完整文件化。

Block Holistic Validity

所有交易跑完之後的最終對帳：

\- 所有 receipt 重新算出 receipts\_root，必須和 header 一致

\- 最終狀態算出 state\_root，必須和 header 一致

\- 累計 gas 不能超過 block gas limit

\- 原子性：沒有「部分接受交易」這回事，全部正確或整個區塊作廢

\---

EL Architecture：客戶端架構

!\[Architecture Overview\]([https://epf.wiki/images/el-architecture/architecture-overview.png](https://epf.wiki/images/el-architecture/architecture-overview.png))

EL 客戶端不只是跑 STF，還要：維護區塊鏈本地副本、透過 DevP2P 和其他 EL 客戶端 gossip、管理交易池、回應 CL 的請求。

核心元件

EVM：虛擬化的 CPU。跟 JVM 的設計動機一樣——不同硬體（x86、ARM、RISC-V）有不同指令集，結果可能不同，所以需要一個虛擬化層確保全網一致。以太坊的 sandwich complexity model：最外層（EVM bytecode）和最上層（Solidity）都盡量簡單，複雜度集中在中間（compiler）。

State：以太坊維護完整的 global state（address → account state 的映射），不像 BTC 只保存 UTXO。State 包含地址、餘額、合約 code/storage、MPT 結構和底層資料庫。

DevP2P：EL 客戶端之間的 P2P 通訊層。交易先存到 mempool，再透過 gossip 傳播。每個收到交易的節點都要先驗證才轉發。

JSON-RPC API：對外接口，錢包和 DApp 透過這個標準 API 查詢狀態或發送交易。

Engine API

Engine API 是 EL 暴露給 CL 的內部介面，不對外公開。用 JSON-RPC over HTTP + JWT 認證（JWT 認證 payload 來源是 CL，但不加密流量）。一個 EL 只能被一個 CL 驅動，但一個 CL 可以連多個 EL 做冗餘。

兩大類 endpoint：

newPayload（V1/V2/V3）：CL 收到新 beacon block 後，抽出 execution payload 丟給 EL 驗證。EL 會：

\- 檢查 parent hash 是否存在且匹配

\- 驗證額外的 execution commitments（Cancun 之後的資料）

\- 執行交易、更新狀態

\- 回傳狀態：VALID / INVALID / SYNCING / ACCEPTED

!\[New Payload Flow\]([https://epf.wiki/images/el-architecture/new-payload.png](https://epf.wiki/images/el-architecture/new-payload.png))

forkChoiceUpdated（V1/V2/V3）：CL 送出 fork choice update（head / safe / finalized block hashes），如果被選為 proposer 還會帶 payload attributes 觸發 block building。EL 會：

\- 更新 canonical head

\- 如果有 payload attributes 就開始 build block

\- 回傳 status + payloadId

!\[Fork Choice Updated Flow\]([https://epf.wiki/images/el-architecture/forkchoice-updated.png](https://epf.wiki/images/el-architecture/forkchoice-updated.png))

啟動流程：CL 先呼叫 engine\_exchangeCapabilities 協商支援的 API 版本 → 發送初始 forkChoiceUpdated → EL 如果還在追趕就回 SYNCING，追上了回 VALID。

Payload Validation

!\[Payload Validation Routine\]([https://epf.wiki/images/el-architecture/payload-validation-routine.png](https://epf.wiki/images/el-architecture/payload-validation-routine.png))

Merge 之後 EL 的角色被大幅簡化。以前 EL 要自己管共識、出塊順序、reorg，現在這些都交給 CL。EL 本質上就是個 STF executor。從 CL 的角度看，整個 EL 就是一個 notify\_new\_payload 的黑盒——丟 payload 進去，回傳 bool。

用 Geth 的簡化 Go 代碼理解 STF：

\`\`\`go

func stf(parent, block, state) (state, error) {

if err := VerifyHeaders(parent, block); err != nil {

return nil, err // header 驗證失敗

}

for \_, tx := range block.Transactions() {

res, err := [vm.Run](http://vm.Run)(block.header(), tx, state)

if err != nil {

return nil, err // 任何一筆交易失敗 → 整個區塊無效

}

state = res // 累積狀態更新

}

return state, nil

}

\`\`\`

header 驗證失敗的例子：gas limit 單一區塊跳太多（超過 parent 的 1/1024）、block number 不連續、1559 base fee 計算不正確。交易執行失敗的話整個區塊直接作廢，不存在部分接受。

同步機制

Full Sync：從 genesis 開始重跑每一個區塊、每一筆交易，重建完整 state trie。最安全但最慢，mainnet 上可能要跑好幾天。注意 EIP-4444 實施後就不再支援從 genesis full sync，會改成 checkpoint sync。

Snap Sync：選一個最近的 finalized block 當 pivot，只下載該 block 的 trie leaves（帳戶和 storage slots）+ Merkle proofs + contract bytecode，在本地重建。之後進入 healing phase 補齊不一致的資料，再從 pivot 之後開始執行交易追到 chain tip。時間從天級降到小時級。

交易池

Legacy Pool：用 price-sorted heap 排序，兩個 heap——一個按 effective tip 排（urgent heap），一個按 gas fee cap 排（floating heap）。飽和時從比較大的 heap 踢交易。

Blob Pool：同樣有 priority heap 但用 logarithmic function 做 eviction，實作文件寫得很詳細（Geth 原始碼裡有大段註解）。

資料庫後端

LevelDB：早期預設，LSM-tree 架構，寫入高吞吐但有 write amplification 問題。已停止維護。

Pebble：Geth 現在的預設後端，同樣 LSM-tree 但改善了 write stall（支援多個 active memtable）、compaction 控制更精細、batch 和 snapshot 語義更強。

MDBX：Erigon 使用，不同的設計哲學（B+tree based），讀取效能好、memory-mapped。

\---

小結

el-specs 把 EL 的行為用數學精確定義，從區塊頭驗證的不等式到 EVM 遞迴執行函數都有形式化描述。el-architecture 則從工程角度拆解客戶端怎麼把這些規範落地——Engine API 怎麼和 CL 對接、sync 怎麼做、交易池怎麼管理、資料怎麼存。

兩篇合在一起看，就是「規範說該怎麼做」和「客戶端實際怎麼做」的完整對照。
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

````
<!-- DAILY_CHECKIN_2026-04-13_END -->
<!-- Content_END -->
