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
<!-- Content_END -->
