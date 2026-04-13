---
timezone: UTC+8
---

# fenixIves

**GitHub ID:** fenixIves

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->
# 一、智能合约核心安全准则

## 1.1 设计层面准则

-   **最小权限原则**：合约功能与权限严格按需分配，避免过度授权。例如，仅核心角色可执行资金转移、合约升级等关键操作，普通用户仅开放必要交互接口。
    
-   **可审计性**：代码逻辑清晰、命名规范，避免冗余嵌套与模糊逻辑，预留日志记录接口，便于后续安全审计与问题追溯。
    
-   **容错性设计**：针对异常场景（如转账失败、权限校验不通过）设计回滚机制，避免资金卡死、状态错乱等问题。
    
-   **拒绝过度复杂**：优先采用成熟、简洁的逻辑实现功能，复杂算法与嵌套逻辑易引入隐藏漏洞，且增加审计难度。
    

## 1.2 开发层面准则

-   **使用安全标准库**：优先采用OpenZeppelin等经社区验证的安全库，避免重复实现核心功能（如ERC20、ERC721标准），减少自定义代码风险。
    
-   **严格类型校验**：对输入参数、状态变量进行类型与范围校验，避免整数溢出/下溢、地址非法等问题。
    
-   **避免硬编码敏感信息**：私钥、API密钥、核心角色地址等敏感信息禁止硬编码，可通过多签机制、参数配置接口动态设置。
    
-   **多版本测试**：在本地测试网（Ganache）、公共测试网（Sepolia、Goerli）进行多场景测试，覆盖正常交互、异常攻击、边界条件等场景。
    

## 1.3 部署与运维层面准则

-   **前置安全审计**：合约部署前必须经过第三方专业审计机构审计，修复所有高、中危漏洞，低危漏洞需评估风险后处理。
    
-   **分步部署策略**：采用“测试网灰度部署→主网小额试点→全量上线”的流程，实时监控合约交互数据，及时发现潜在问题。
    
-   **应急响应机制**：预留合约暂停、升级、资金紧急提取接口（需严格权限控制），针对黑客攻击、漏洞爆发等突发情况可快速处置。
    

# 二、常见漏洞类型、代码示例与防护方案

以下漏洞以主流智能合约语言Solidity为例，涵盖DeFi、NFT等场景高频漏洞，同时提供风险代码与安全代码对比。

## 2.1 整数溢出/下溢漏洞

### 2.1.1 漏洞原理

Solidity早期版本（<0.8.0）未内置整数溢出/下溢检查，当整数运算结果超出其数据类型范围时，会出现异常值。例如，uint256最大值为2²⁵⁶-1，若在此基础上加1，结果会变为0（溢出）；uint256最小值为0，减1会变为2²⁵⁶-1（下溢）。

### 2.1.2 风险代码示例

```
// Solidity 0.7.6（无内置溢出检查）
pragma solidity ^0.7.6;

contract OverflowDemo {
    uint256 public balance;

    // 存款函数，存在溢出风险
    function deposit(uint256 amount) public {
        balance += amount; // 当balance + amount > 2^256-1时，发生溢出，balance值异常
    }

    // 取款函数，存在下溢风险
    function withdraw(uint256 amount) public {
        balance -= amount; // 当balance < amount时，发生下溢，balance变为极大值
    }
}
```

### 2.1.3 防护方案

-   升级Solidity版本至0.8.0及以上，该版本内置溢出/下溢检查，触发时会自动回滚交易。
    
-   早期版本可使用OpenZeppelin的SafeMath库进行运算校验。
    

### 2.1.4 安全代码示例

```
// 方案1：使用Solidity 0.8.0+内置检查
pragma solidity ^0.8.19;

contract SafeMathDemo1 {
    uint256 public balance;

    function deposit(uint256 amount) public {
        balance += amount; // 溢出时自动回滚
    }

    function withdraw(uint256 amount) public {
        require(balance >= amount, "Insufficient balance"); // 额外增加逻辑校验
        balance -= amount; // 下溢时自动回滚
    }
}

// 方案2：早期版本使用SafeMath（Solidity <0.8.0）
pragma solidity ^0.7.6;
import "@openzeppelin/contracts/math/SafeMath.sol";

contract SafeMathDemo2 {
    using SafeMath for uint256;
    uint256 public balance;

    function deposit(uint256 amount) public {
        balance = balance.add(amount); // SafeMath.add自动检查溢出
    }

    function withdraw(uint256 amount) public {
        balance = balance.sub(amount); // SafeMath.sub自动检查下溢，不足时抛异常
    }
```

## 2.2 重入攻击漏洞（Reentrancy）

### 2.2.1 漏洞原理

当合约A调用合约B的函数时，合约B可在执行过程中再次调用合约A的敏感函数（如资金转移函数），形成递归调用。若合约A未先更新状态再执行外部调用，攻击者可利用该漏洞重复提取资金。

### 2.2.2 风险代码示例

```
pragma solidity ^0.8.19;

contract ReentrancyRisk {
    mapping(address => uint256) public userBalances;

    // 用户存款
    function deposit() public payable {
        userBalances[msg.sender] += msg.value;
    }

    // 取款函数，存在重入风险
    function withdraw() public {
        uint256 amount = userBalances[msg.sender];
        require(amount > 0, "No balance");

        // 先执行外部转账，再更新状态
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        // 状态更新在外部调用之后，攻击者可重复调用withdraw
        userBalances[msg.sender] = 0;
    }

    // 接收ETH的回退函数
    receive() external payable {}
}
```

攻击逻辑：攻击者部署恶意合约，先向ReentrancyRisk存款，再调用withdraw。恶意合约的fallback/receive函数会再次调用ReentrancyRisk的withdraw，此时userBalances尚未置零，可重复提取资金。

### 2.2.3 防护方案

-   **Checks-Effects-Interactions 模式**：先执行权限/余额校验（Checks），再更新合约状态（Effects），最后执行外部调用（Interactions）。
    
-   使用OpenZeppelin的ReentrancyGuard库，通过锁机制禁止重入调用。
    

### 2.2.4 安全代码示例

```
pragma solidity ^0.8.19;
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

// 继承ReentrancyGuard启用重入保护
contract SafeReentrancy is ReentrancyGuard {
    mapping(address => uint256) public userBalances;

    function deposit() public payable {
        userBalances[msg.sender] += msg.value;
    }

    // 使用nonReentrant修饰符禁止重入
    function withdraw() public nonReentrant {
        uint256 amount = userBalances[msg.sender];
        // Checks：校验余额
        require(amount > 0, "No balance");

        // Effects：先更新状态，再执行外部调用
        userBalances[msg.sender] = 0;

        // Interactions：执行转账
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }

    receive() external payable {}
}
```

## 2.3 访问控制漏洞

### 2.3.1 漏洞原理

合约未严格控制关键函数的访问权限，导致普通用户可执行管理员操作（如修改参数、提取合约资金、暂停合约等）。常见场景包括：未校验权限、权限逻辑错误、过度授权。

### 2.3.2 风险代码示例

```
pragma solidity ^0.8.19;

contract AccessControlRisk {
    address public owner;
    uint256 public feeRate;

    constructor() {
        owner = msg.sender; // 部署者为管理员
    }

    // 无权限校验，任何人可修改费率
    function setFeeRate(uint256 newRate) public {
        feeRate = newRate;
    }

    // 权限校验逻辑错误（使用==而非===，虽Solidity中地址比较无差异，但逻辑不严谨，且无异常处理）
    function withdrawFunds() public {
        if (msg.sender == owner) {
            payable(owner).transfer(address(this).balance);
        }
    }
}
```

### 2.3.3 防护方案

-   使用OpenZeppelin的Ownable、AccessControl库实现精细化权限管理。
    
-   关键函数必须添加权限修饰符，明确可执行角色。
    
-   避免硬编码角色地址，支持角色转让、新增功能，且操作需留痕。
    

### 2.3.4 安全代码示例

```
pragma solidity ^0.8.19;
import "@openzeppelin/contracts/access/Ownable.sol";

// 继承Ownable实现管理员权限控制
contract SafeAccessControl is Ownable {
    uint256 public feeRate;

    // 仅管理员可调用的修饰符（Ownable已内置onlyOwner修饰符）
    function setFeeRate(uint256 newRate) public onlyOwner {
        require(newRate > 0 && newRate <= 1000, "Invalid rate (0-1000)"); // 额外校验参数合法性
        feeRate = newRate;
    }

    // 提取资金，仅管理员可执行
    function withdrawFunds() public onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds to withdraw");
        payable(owner()).transfer(balance);
    }

    // 接收ETH
    receive() external payable {}
}
```

## 2.4 \_front-run攻击漏洞

### 2.4.1 漏洞原理

由于区块链交易的公开性与确认延迟，攻击者可通过监控待打包交易池，利用更高的Gas费抢先打包自己的交易，操纵交易执行顺序获利。常见于DeFi交易、NFT mint、拍卖等场景。

### 2.4.2 风险代码示例

```
pragma solidity ^0.8.19;

contract FrontRunRisk {
    // 交易对价格映射
    mapping(address => mapping(address => uint256)) public tokenPrice;

    // 设置代币价格，存在抢先交易风险
    function setTokenPrice(address tokenA, address tokenB, uint256 price) public {
        tokenPrice[tokenA][tokenB] = price;
    }

    // 根据设定的价格兑换代币
    function swap(address tokenA, address tokenB, uint256 amount) public {
        uint256 cost = amount * tokenPrice[tokenA][tokenB];
        // 兑换逻辑（省略代币转账细节）
    }
}
```

攻击逻辑：攻击者监控到用户调用setTokenPrice设置低价后，立即以更高Gas费调用setTokenPrice修改为高价，再调用swap兑换，导致原用户交易执行时价格异常，遭受损失。

### 2.4.3 防护方案

-   采用批量交易机制，将关键操作（如价格设置、兑换）打包为原子交易，避免中间被插针。
    
-   引入时间锁机制，关键参数修改后需等待一定时间（如24小时）生效，给用户反应时间。
    
-   使用随机化交易执行顺序，或基于链上随机数（需安全实现）避免交易顺序被操纵。
    

### 2.4.4 安全代码示例

```
pragma solidity ^0.8.19;

contract SafeFrontRun {
    mapping(address => mapping(address => uint256)) public tokenPrice;
    mapping(address => mapping(address => uint256)) public pendingPrice;
    mapping(address => mapping(address => uint256)) public priceEffectiveTime;

    uint256 public constant TIME_LOCK = 1 days; // 时间锁：1天

    // 提交待生效价格，触发时间锁
    function proposeTokenPrice(address tokenA, address tokenB, uint256 price) public {
        pendingPrice[tokenA][tokenB] = price;
        priceEffectiveTime[tokenA][tokenB] = block.timestamp + TIME_LOCK;
    }

    // 时间锁到期后生效价格
    function confirmTokenPrice(address tokenA, address tokenB) public {
        require(block.timestamp >= priceEffectiveTime[tokenA][tokenB], "Time lock not expired");
        tokenPrice[tokenA][tokenB] = pendingPrice[tokenA][tokenB];
        // 清空待生效记录
        pendingPrice[tokenA][tokenB] = 0;
        priceEffectiveTime[tokenA][tokenB] = 0;
    }

    function swap(address tokenA, address tokenB, uint256 amount) public {
        uint256 cost = amount * tokenPrice[tokenA][tokenB];
        // 兑换逻辑（省略代币转账细节）
    }
}
```

## 2.5 恶意代码注入漏洞

### 2.5.1 漏洞原理

合约接收外部传入的代码片段（如calldata、函数签名）并执行，攻击者可构造恶意代码注入，操纵合约状态、提取资金。常见于动态调用（call、delegatecall）场景。

### 2.5.2 风险代码示例

```
pragma solidity ^0.8.19;

contract CodeInjectionRisk {
    // 动态执行外部传入的代码
    function execute(address target, bytes calldata data) public payable {
        // 无任何校验，直接执行目标合约的任意函数
        (bool success, ) = target.call{value: msg.value}(data);
        require(success, "Execution failed");
    }
}
```

攻击逻辑：攻击者调用execute函数，传入恶意合约地址和函数签名，让CodeInjectionRisk合约以自身权限执行恶意代码，如提取合约资金、修改状态变量。

### 2.5.3 防护方案

-   禁止无限制动态调用，仅允许调用白名单内的合约地址与函数。
    
-   对传入的calldata进行校验，验证函数签名合法性。
    
-   避免使用delegatecall（会将目标合约代码在当前合约上下文执行，风险极高），必须使用时需严格控制目标合约权限。
    

### 2.5.4 安全代码示例

```
pragma solidity ^0.8.19;

contract SafeCodeExecution {
    // 可调用的合约白名单
    mapping(address => bool) public allowedTargets;
    // 可执行的函数签名白名单（以bytes4表示）
    mapping(bytes4 => bool) public allowedFunctions;

    constructor() {
        // 初始化白名单（示例：允许调用USDT合约的transfer函数）
        allowedFunctions[bytes4(keccak256("transfer(address,uint256)"))] = true;
    }

    // 仅管理员可添加白名单
    function addAllowedTarget(address target) public onlyOwner {
        allowedTargets[target] = true;
    }

    // 安全执行外部调用
    function safeExecute(address target, bytes calldata data) public payable onlyOwner {
        // 校验目标合约在白名单内
        require(allowedTargets[target], "Target not allowed");
        // 提取函数签名（前4字节）
        bytes4 funcSig = bytes4(data[:4]);
        // 校验函数签名在白名单内
        require(allowedFunctions[funcSig], "Function not allowed");

        (bool success, ) = target.call{value: msg.value}(data);
        require(success, "Execution failed");
    }
}
```

# 三、额外安全建议

1.  **依赖库安全**：定期更新OpenZeppelin等依赖库，关注社区漏洞通报，及时修复依赖项中的安全问题。
    
2.  **链上监控**：部署后通过链上监控工具（如Etherscan、Nansen）实时跟踪合约交易，异常资金流动、高频调用需及时排查。
    
3.  **隐私保护**：避免在合约中存储敏感数据（如用户隐私信息），链上数据公开可查，敏感信息需离线加密存储。
    
4.  **持续学习**：区块链安全技术迭代迅速，需关注最新漏洞案例与防护方案，定期开展内部安全培训。
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->

# 以太坊 JSON-RPC 详细学习笔记（移动端友好版）

核心一句话：**JSON-RPC 是以太坊生态的通用通信协议，是用户、钱包、DApp、共识层与以太坊节点/执行层交互的唯一标准方式**，基于JSON格式的远程过程调用规范，属于以太坊执行API的核心组成部分。

* * *

## 一、JSON-RPC 核心本质与作用

### 1\. 基础定义

JSON-RPC 是基于 OpenRPC 规范、使用 JSON 格式编码的远程过程调用协议。它允许客户端向远程以太坊节点发起函数调用，并获取执行结果，是以太坊执行层API的核心规范。

### 2\. 核心应用场景

-   用户/钱包/DApp 与以太坊节点交互（查余额、发交易、读合约、监听事件等）
    
-   以太坊共识层（CL）与执行层（EL）之间的内部通信（专属 Engine API）
    
-   区块浏览器、链上数据分析工具、自动化脚本获取链上原始数据
    
-   节点管理员对节点进行配置、调试、管理操作
    

### 3\. 核心特性

-   **规范统一**：所有以太坊客户端（Geth、Reth、Erigon等）必须实现标准方法，保证跨客户端行为完全一致
    
-   **可扩展**：支持客户端自定义专属方法，满足节点的个性化管理、调试需求
    
-   **传输无关**：不绑定底层传输协议，可基于HTTP、WebSocket、IPC等多种协议使用
    

* * *

## 二、JSON-RPC 请求的标准结构

所有以太坊 JSON-RPC 请求必须遵循统一的JSON格式，无论哪种客户端、哪种传输协议，结构完全固定。

### 标准请求示例

```
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "<命名空间_方法名>",
  "params": []
}
```

### 字段全解析

1.  **id**：请求的唯一标识符，用于匹配请求和对应的响应，避免并发请求的响应错乱。同一客户端的并发请求需使用不同id。
    
2.  **jsonrpc**：JSON-RPC 协议版本，以太坊所有标准方法固定为`2.0`。
    
3.  **method**：要调用的目标方法名，格式固定为`命名空间_方法名`，前缀为命名空间，通过下划线与方法名分隔，例如`eth_getBalance`。
    
4.  **params**：方法的入参数组，无参数时必须传空数组`[]`；部分方法支持默认参数，不传时会使用协议默认值。
    

### 响应格式说明

-   成功响应：返回对应`id`、`jsonrpc`和`result`字段，`result`为方法执行结果
    
-   失败响应：返回对应`id`、`jsonrpc`和`error`字段，`error`包含错误码和错误信息
    

* * *

## 三、核心命名空间与常用方法

所有方法都通过「命名空间前缀」分类，分为**标准必选方法**（所有以太坊客户端必须实现）和**客户端自定义方法**（不同客户端的专属扩展）。

### 命名空间总览

| 命名空间 | 核心用途 | 敏感等级 |
| eth | 以太坊核心链上交互（查余额、发交易、读合约等） | 部分敏感 |
| web3 | web3客户端工具类方法 | 非敏感 |
| net | 节点网络信息查询 | 非敏感 |
| txpool | 交易内存池查询 | 非敏感 |
| debug | 链上状态深度调试、Geth风格追踪 | 非敏感（公共RPC常限制） |
| trace | Parity风格的交易追踪、状态回溯 | 非敏感（公共RPC常限制） |
| admin | 节点配置、管理操作 | 高敏感 |
| rpc | RPC服务本身的信息查询 | 非敏感 |

### 1\. eth 命名空间（最常用，核心必学）

这是所有DApp、钱包最核心依赖的命名空间，提供以太坊网络的基础访问能力，覆盖90%以上的日常开发场景。

| 方法名 | 核心入参 | 核心作用 |
| eth_blockNumber | 无 | 获取当前最新区块高度 |
| eth_chainId | 无 | 获取当前链的chainId（交易防重放核心参数） |
| eth_getBalance | 地址、区块号 | 查询指定地址的ETH余额 |
| eth_getCode | 合约地址、区块号 | 查询指定地址的合约字节码 |
| eth_getStorageAt | 合约地址、存储槽位、区块号 | 查询合约指定存储槽位的值 |
| eth_gasPrice | 无 | 获取当前网络的建议gas单价（单位：wei） |
| eth_estimateGas | 交易对象 | 模拟执行交易，估算所需gas量 |
| eth_call | 交易对象 | 模拟执行合约调用，不上链、不消耗gas |
| eth_sendRawTransaction | 签名后的交易原始数据 | 广播已签名的交易到以太坊网络 |
| eth_getBlockByNumber | 区块号、是否返回完整交易 | 按区块号查询区块详情 |
| eth_getBlockByHash | 区块哈希、是否返回完整交易 | 按区块哈希查询区块详情 |
| eth_getLogs | 过滤器对象 | 查询匹配过滤条件的合约事件日志 |
| eth_getTransactionByHash | 交易哈希 | 按交易哈希查询交易详情 |
| eth_getTransactionReceipt | 交易哈希 | 查询交易的执行回执（状态、gas消耗、日志等） |

### 2\. debug 命名空间

提供链上原始数据的深度访问能力，多用于区块浏览器、合约审计、链上数据分析。  
⚠️ 注意：公共RPC节点大多会限制或禁用该命名空间，因为部分方法会消耗节点大量计算资源，且非归档节点无法查询历史状态。

常用核心方法：

-   debug\_getBadBlocks：获取节点收到的无效坏区块列表
    
-   debug\_getRawBlock：获取RLP编码的原始区块数据
    
-   debug\_getRawHeader：获取RLP编码的区块头数据
    
-   debug\_getRawReceipts：获取指定区块内二进制编码的交易回执
    
-   debug\_getRawTransactions：获取二进制编码的原始交易数据
    

### 3\. 其他常用非敏感命名空间

-   **web3**：工具类方法，最常用`web3_sha3`，用于计算数据的keccak256哈希
    
-   **net**：节点网络信息查询，常用`net_version`（网络ID）、`net_peerCount`（节点连接的peer数量）
    
-   **txpool**：交易内存池查询，常用`txpool_content`（内存池内的所有交易）、`txpool_status`（内存池交易数量统计）
    
-   **rpc**：RPC服务本身查询，常用`rpc_modules`（查询当前节点开启的所有RPC命名空间）
    

### 4\. 高敏感命名空间（admin）

用于节点的本地配置和管理，仅节点管理员可访问，公共节点绝对不会开放。核心用途包括添加节点peer、设置节点配置、管理节点运行状态等，暴露到公网会导致节点被恶意控制。

* * *

## 四、特殊的 Engine API

Engine API 是以太坊合并后，专门为共识层（CL）和执行层（EL）通信设计的专属JSON-RPC接口，与普通用户使用的RPC完全隔离。

### 核心特点

1.  **非用户面向**：仅用于CL和EL之间的内部通信，普通用户/DApp无法访问
    
2.  **独立端点**：执行客户端在单独的、带认证的端口提供Engine API，不与普通HTTP RPC端口共用
    
3.  **安全认证**：通过JWT（JSON Web Token）认证，只有合法的共识层客户端可以访问
    
4.  **核心作用**：负责共识层与执行层之间的共识信息交换、分叉选择更新、区块验证、执行载荷构建等核心共识流程
    

### 核心方法

| 方法名 | 核心作用 |
| engine_exchangeTransitionConfigurationV1 | 共识层与执行层交换配置信息，保证两端配置一致 |
| engine_forkchoiceUpdatedV1 | 更新分叉选择状态，可选触发区块执行载荷的构建 |
| engine_getPayloadV1 | 获取执行层已构建完成的区块执行载荷 |
| engine_newPayloadV1 | 验证新的区块执行载荷的合法性 |

补充：带\*标记的方法有多个版本，用于适配以太坊网络升级和协议迭代，保证向后兼容。

* * *

## 五、参数编码规范（必记，踩坑重灾区）

以太坊JSON-RPC有严格的参数编码规则，不遵守会直接导致请求失败，核心规则为：**所有数值、哈希、地址、字节数据，都必须使用带0x前缀的十六进制字符串表示**。

### 1\. 数值类（Quantities）

-   规则：整数必须转为带`0x`前缀的十六进制字符串，禁止前导零（数字0除外）
    
-   正确示例：
    

-   数字65 → `"0x41"`
    
-   数字0 → `"0x0"`
    
-   1 ETH（1e18 wei）→ `"0xde0b6b3a7640000"`
    

-   错误示例：
    

-   `"0x"`（无后续数字）
    
-   `"ff"`（无0x前缀）
    
-   `"0x041"`（多余的前导零）
    

### 2\. 未格式化数据类（Unformatted Data）

-   覆盖范围：地址、区块哈希、交易哈希、字节数组、合约字节码等
    
-   规则：必须转为带`0x`前缀的十六进制字符串，固定长度的类型必须保持对应字符数
    
-   正确示例：
    

-   20字节地址 → `"0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266"`
    
-   32字节哈希 → `"0xdfaf2817f19963846490b330ae33eba7b42872e8c8bd111c8d7ea3846c84cd"`
    

-   错误示例：无前缀、长度不符、包含非十六进制字符
    

* * *

## 六、传输协议（Transport）

JSON-RPC 是**传输无关**的协议，可基于多种传输层协议使用，不同协议有明确的适用场景。

| 传输协议 | 通信模式 | 核心特点 | 适用场景 |
| HTTP | 单向请求-响应 | 连接在响应返回后关闭，简单通用，兼容性最好 | 绝大多数DApp、钱包的单次链上数据查询、交易广播 |
| WebSocket（WSS） | 双向长连接 | 连接持续保持，支持订阅式事件推送 | 合约事件监听、新区块通知、实时价格推送等需要持续数据更新的场景 |
| IPC（进程间通信） | 本地进程双向通信 | 速度最快、安全性最高，仅本机可用 | 本地节点的控制台交互、本地服务与节点的高频通信 |

* * *

## 七、实操工具与代码示例

### 1\. curl 命令行（最简单，快速测试）

无需安装额外依赖，可快速验证RPC节点可用性、测试方法调用。

示例1：获取最新区块高度

```
curl <你的节点RPC地址> \
-X POST \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_blockNumber",
  "params":[],
  "id":1
}'
```

示例2：查询地址余额

```
curl <你的节点RPC地址> \
-X POST \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_getBalance",
  "params":["0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266", "latest"],
  "id":1
}'
```

### 2\. JS/TS 原生调用（axios）

前端/Node.js环境直接发起请求，无需依赖web3库。

```
import axios from 'axios';

// 节点RPC地址
const RPC_ENDPOINT = '<你的节点RPC地址>';
// 要查询的地址
const ADDRESS = '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266';

// 发起JSON-RPC请求
const getBalance = async () => {
  const response = await axios.post(RPC_ENDPOINT, {
    jsonrpc: '2.0',
    method: 'eth_getBalance',
    params: [ADDRESS, 'latest'],
    id: 1
  }, {
    headers: { 'Content-Type': 'application/json' }
  });
  console.log('余额（wei）：', response.data.result);
};

getBalance();
```

### 3\. Web3 库（开发首选，封装友好）

主流web3库都封装了JSON-RPC方法，无需手动构建请求体，是日常开发的首选。

Python（web3py）

```
from web3 import Web3

# 连接节点
w3 = Web3(Web3.HTTPProvider('http://localhost:8545'))

# 调用封装后的JSON-RPC方法
balance = w3.eth.get_balance('0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266')
block_number = w3.eth.block_number

print(f'最新区块高度：{block_number}')
print(f'地址余额：{balance} wei')
```

JS/TS（ethers.js v6）

```
import { ethers } from "ethers";

// 连接节点
const provider = new ethers.JsonRpcProvider('http://localhost:8545');

// 调用封装后的方法
const blockNumber = await provider.getBlockNumber();
const balance = await provider.getBalance('0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266');

console.log('最新区块高度：', blockNumber);
console.log('地址余额：', balance.toString(), 'wei');
```

* * *

## 八、关键注意事项与避坑指南

1.  **公共RPC节点限制**  
    公共免费RPC节点（Infura、Alchemy免费层、社区公共节点）大多会限制请求频率，禁用debug、trace等高消耗命名空间；高频率的链上数据查询，建议使用付费RPC服务或自建节点。
    
2.  **区块号参数规范**  
    所有需要传区块号的方法，除了十六进制的区块高度，还可以使用标准标签：
    

-   `latest`：最新已打包的区块（最常用默认值）
    
-   `earliest`：创世区块
    
-   `pending`：内存池中的待打包区块
    

3.  **安全风险**
    

-   绝对不要把带admin、eth\_sendTransaction等敏感方法的RPC端口暴露到公网，会导致节点被攻击、资产被盗
    
-   不要信任未知的第三方RPC节点，恶意节点可能返回虚假数据、篡改交易内容
    
-   Engine API的JWT密钥必须严格保密，仅允许共识层与执行层访问
    

4.  **方法兼容性**
    

-   不同以太坊客户端的自定义方法可能不通用，但标准方法完全兼容
    
-   部分方法有多个版本，需注意对应以太坊的网络升级版本，避免调用失效
    

5.  **模拟执行方法特性**  
    `eth_call`和`eth_estimateGas`是链下模拟执行，不会上链、不消耗gas，可用于合约调用测试、gas估算，不会改变链上状态。
    

* * *

## 核心速记清单

-   JSON-RPC是以太坊的**通用通信标准**，所有链上交互都依赖它
    
-   请求固定4字段：id、jsonrpc=2.0、method、params
    
-   方法名固定格式：`命名空间_方法名`，eth是最核心的常用命名空间
    
-   所有参数必须用**0x前缀的十六进制字符串**，禁止无前缀、禁止多余前导零
    
-   普通单次查询用HTTP，实时订阅用WSS，本地节点用IPC
    
-   Engine API是CL-EL通信专用，与用户RPC隔离，带JWT认证
    
-   开发优先用web3/ethers等封装库，无需手动构建请求体
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->


# 以太坊交易（Transaction）详细学习笔记

核心一句话：**交易是以太坊上唯一能触发链上状态变更的、由外部账户签名的加密指令**，是连接用户与EVM、实现所有链上操作的唯一载体。

* * *

## 一、交易的核心本质与基础属性

1.  **发起方限制**：只能由\*\*外部账户（EOA，即用户钱包）\*\*发起，合约账户无法主动发起交易，仅能被动被调用后触发内部消息
    
2.  **安全保障**：必须经过发送方的ECDSA加密签名，防篡改、防冒充、防重放
    
3.  **传播方式**：通过JSON-RPC协议广播到以太坊全网节点
    
4.  **最终作用**：触发EVM执行代码，完成以太坊世界状态的确定性变更，最终由矿工打包进区块永久上链
    

* * *

## 二、交易的7大核心字段（全详解）

交易的所有行为、类型、规则，都由这7个字段决定，每个字段都有不可替代的作用：

### 1\. nonce（交易序号）

-   **基础定义**：等于发送方地址已经发出的交易总数的整数值，从0开始依次递增
    
-   **核心作用1：防重放攻击**  
    每笔交易对应唯一nonce，同一笔交易无法被重复广播执行。比如Alice给Bob转1ETH，Bob无法重复广播这笔交易再次扣款，因为EVM会拒绝相同nonce的已执行交易
    
-   **核心作用2：计算合约地址**  
    部署合约时，新合约的地址 = 发送方地址 + nonce 做哈希计算得出，可提前预测
    
-   **核心作用3：加速/取消 pending 交易**  
    交易因gasPrice过低卡在内存池时，可发送**相同nonce、更高gasPrice**的新交易，覆盖原交易；取消交易则发送同nonce、0转账、更高gasPrice的交易。  
    ⚠️ 注意：覆盖成功率取决于矿工打包规则和网络状态，并非100%成功
    
-   **关键规则**：EVM会严格按nonce从小到大执行交易，nonce不连续的交易会卡在内存池，不会被打包
    

### 2\. gasPrice（gas单价）

-   **基础定义**：发送方愿意为每单位gas支付的费用，单位是Wei（以太坊最小单位）
    
-   **换算规则**：1 ETH = 10¹⁸ Wei
    
-   **核心作用**：决定交易的打包优先级，gasPrice越高，矿工越优先将这笔交易打包进区块
    
-   **补充说明**：伦敦升级后，新增了baseFee（基础费）和maxPriorityFee（小费），但gasPrice的核心定价逻辑不变
    

### 3\. gasLimit（gas上限）

-   **基础定义**：发送方为这笔交易设置的最大可消耗gas数量
    
-   **核心作用**：防止交易执行中出现无限循环、异常逻辑耗尽节点资源。交易执行中gas耗尽时，会立即终止执行，所有状态回滚，已消耗的gas不予退还
    
-   **补充说明**：未消耗完的gas，会在交易执行完成后退还给发送方
    

### 4\. to（接收方地址）

-   **基础定义**：20字节的以太坊接收地址，**这个字段直接决定了交易的类型和执行逻辑**
    
-   **三种取值对应三种交易模式**：
    

| to字段取值 | 交易模式 | 核心行为 |
| 空值（null） | 合约创建 | 部署一个全新的智能合约 |
| 外部账户（EOA）地址 | 价值转移 | 单纯的ETH转账，无合约代码执行 |
| 合约账户地址 | 合约执行 | 调用已部署合约的代码，执行合约逻辑 |

### 5\. value（转账金额）

-   **基础定义**：本次交易向接收方地址转移的Wei数量
    
-   **不同交易模式的作用**：
    

-   纯转账交易：就是给接收方转的ETH数量
    
-   合约创建交易：作为新部署合约的初始余额
    
-   合约调用交易：给目标合约转入的ETH数量，常用于支付mint、swap等操作的费用
    

### 6\. data / init（EVM输入数据）

-   **基础定义**：无长度限制的字节数组，是给EVM执行的核心输入数据，在不同交易模式下有完全不同的含义
    
-   **合约创建模式**：该字段称为`init bytecode`，由两部分组成：
    

1.  初始化代码：仅在合约部署时执行一次，核心作用是复制运行时代码到内存并返回
    
2.  运行时字节码：部署完成后永久存在合约账户里的代码，后续每次调用合约都会执行这段代码
    

-   **合约调用模式**：该字段称为`input data`，包含要调用的合约函数选择器，以及函数对应的入参编码
    
-   **纯转账交易**：该字段可留空，也可附加备注信息
    

### 7\. signature（签名，v/r/s）

-   **基础定义**：发送方对交易的ECDSA签名，由v、r、s三个值组成
    
-   **核心作用**：
    

1.  证明这笔交易确实是私钥持有者发起的，不可冒充
    
2.  证明交易内容在签名后没有被篡改
    
3.  可通过签名反推出交易的发送方地址（from字段）
    

* * *

## 三、三大交易类型的完整执行流程

### 1\. 纯ETH转账交易（to=EOA地址）

这是最简单的交易类型，执行流程：

1.  钱包构建交易，填充nonce、gasPrice、gasLimit、to=接收方钱包地址、value=转账金额、data留空
    
2.  用私钥对交易签名，广播到全网节点
    
3.  矿工验证签名、nonce、账户余额是否足够支付转账+gas费用
    
4.  验证通过后，EVM执行状态变更：从发送方地址扣减ETH+gas费用，给接收方地址增加对应ETH
    
5.  交易打包进区块，生成交易回执，状态标记为成功（1）
    

### 2\. 合约创建交易（to=空值）

这是部署智能合约的唯一方式，核心是init代码的执行，流程：

1.  钱包构建交易，to字段留空，data字段填充完整的init bytecode（初始化代码+运行时代码）
    
2.  签名后广播全网，矿工验证交易合法性
    
3.  EVM执行init代码：
    

-   第一步：通过CODECOPY指令，把init代码里的运行时字节码复制到内存
    
-   第二步：通过RETURN指令，返回内存里的运行时字节码
    

4.  EVM用发送方地址+nonce计算出新合约的地址，将返回的运行时字节码永久存储到该合约地址
    
5.  交易打包上链，回执里会返回新合约的地址，合约部署完成
    
6.  后续所有对该合约的调用，都会执行存储在合约账户里的运行时字节码
    

### 3\. 合约调用交易（to=合约地址）

这是最常用的链上交互方式（swap、mint、质押等都属于这类），流程：

1.  钱包构建交易，to字段填目标合约地址，data字段填充要调用的函数选择器+入参编码
    
2.  签名后广播全网，矿工验证交易合法性
    
3.  EVM加载目标合约地址里存储的运行时字节码，按data字段的指令执行对应函数逻辑
    
4.  执行过程中，EVM会完成对应的计算、存储读写、事件触发、跨合约调用等操作
    
5.  执行完成后，所有状态变更同步到以太坊世界状态，交易打包上链，生成回执
    

* * *

## 四、交易回执（Receipt）：交易执行的“结果单”

每一笔交易执行完成后，无论成功还是失败，都会生成对应的交易回执，它是EVM状态转换的输出产物，永久存储在区块里。

### 回执的5大核心字段

1.  **交易类型**：区分是legacy传统交易，还是EIP-2718后的类型化交易
    
2.  **状态（Status）**：交易执行结果，只有两个取值
    

-   1 = 交易执行成功，所有状态变更已生效
    
-   0 = 交易执行失败，所有状态变更回滚，已消耗gas不予退还
    

3.  **Gas Used**：区块内该交易之前所有交易的累计gas消耗 + 本次交易的gas消耗
    
4.  **日志（Logs）**：合约执行中触发的事件记录，由合约地址、索引topic、非索引原始数据组成，是前端、链下服务监听合约事件的核心依据
    
5.  **Logs Bloom**：256字节的布隆过滤器，用于快速检索区块内的日志，不用遍历整个区块就能快速判断某个地址/事件是否在区块日志里
    

### 补充说明

回执会被编码后，存入区块的\*\*回执默克尔树（Receipt Trie）\*\*中，用于区块共识和轻节点验证。

* * *

## 五、交易格式的演进：EIP-2718 类型化交易

### 1\. 传统交易（Legacy Transaction）的痛点

在EIP-2718之前，所有交易都用RLP编码，新增交易类型需要用复杂的兼容方案，设计脆弱，扩展性极差。

### 2\. EIP-2718 类型化交易的核心设计

EIP-2718引入了“类型化信封”格式，让交易和回执的扩展变得简单，同时完全向后兼容传统交易。

格式规则

-   类型化交易格式：`Typed Transaction = 交易类型单字节 + 交易负载`
    
-   类型化回执格式：`Typed Receipt = 回执类型单字节 + 回执负载`
    
-   回执的类型字节，必须和对应交易的类型字节完全一致，保证客户端可确定性解码
    

快速识别规则

-   交易/回执的首字节在`0x00-0x7f`之间：是类型化交易/回执
    
-   交易/回执的首字节≥`0xc0`：是传统RLP编码的legacy交易/回执
    

核心优势

-   可扩展性极强，新增交易类型只需新增一个类型字节，无需修改原有编码逻辑
    
-   完全向后兼容，传统交易可正常执行
    
-   编码更简洁，解码更高效，安全性更高
    

* * *

## 六、交易的完整生命周期与签名原理

### 1\. 交易从发起至上链的完整生命周期

1.  **交易构建**：钱包/代码填充交易的7大核心字段，完成交易体的组装
    
2.  **交易签名**：用发送方的私钥对交易体进行加密签名，生成v/r/s签名值，组装成完整的已签名交易
    
3.  **全网广播**：通过JSON-RPC将已签名交易广播到以太坊节点，进入节点的内存池
    
4.  **矿工验证**：矿工验证交易的签名、nonce、余额、gas等是否合法，过滤无效交易
    
5.  **打包排序**：矿工按gasPrice优先级对合法交易排序，打包进候选区块
    
6.  **EVM执行**：矿工按顺序执行区块内的每一笔交易，更新世界状态，生成交易回执
    
7.  **共识上链**：区块通过全网共识后，被添加到区块链主网，交易最终确认，状态永久生效
    

### 2\. 交易签名的核心步骤（对应文档sign.js）

交易签名是保障安全的核心，完整步骤如下：

1.  **RLP编码**：将组装好的交易体，用RLP编码成字节数组
    
2.  **哈希计算**：对RLP编码后的交易体，做keccak256哈希，得到交易消息哈希
    
3.  **ECDSA签名**：用发送方的32字节私钥，对消息哈希做ECDSA签名，生成v、r、s三个签名值
    
4.  **组装完整交易**：将v、r、s签名值追加到原始交易体中
    
5.  **最终编码**：对追加了签名的完整交易体，再次做RLP编码，得到最终可广播的十六进制交易字符串
    

* * *

## 七、实操案例全拆解（Foundry本地部署+调用）

文档中用Foundry工具完成了合约部署和调用，这里拆解每一步的核心逻辑：

### 案例背景

要部署一个合约，功能是：执行时计算6\*7=42，将结果存入合约的0号存储槽。

### 步骤1：准备合约字节码

合约的运行时代码汇编：

```
[00] PUSH1 06  // 把6压入栈
[02] PUSH1 07  // 把7压入栈
[04] MUL        // 栈顶两个值相乘，得到42
[05] PUSH1 00  // 把存储槽0压入栈
[07] SSTORE     // 把42存入0号存储槽
```

对应的运行时字节码：`6006600702600055`

### 步骤2：构建init字节码

init代码需要完成“复制运行时代码到内存并返回”的逻辑，最终init字节码为：  
`6008600c60003960086000f36006600702600055`

### 步骤3：构建部署交易体

```
[
  "0x", // nonce=0，发送方的第一笔交易
  "0x77359400", // gasPrice=2Gwei
  "0x13880", // gasLimit=80000，合约部署标准值
  "0x", // to字段为空，代表合约创建
  "0x05", // value=5wei，给新合约的初始余额
  "0x6008600c60003960086000f36006600702600055" // init代码
]
```

### 步骤4：签名并广播交易

用anvil本地节点的测试私钥对交易签名，通过cast publish广播到本地节点，交易上链后，回执返回新合约地址：`0x5fbdb2315678afecb367f032d93f642f64180aa3`

### 步骤5：验证部署结果

-   查合约代码：`cast code 合约地址`，返回运行时字节码`6006600702600055`，部署成功
    
-   查合约余额：`cast balance 合约地址`，返回5，初始余额到账
    

### 步骤6：调用合约执行逻辑

构建合约调用交易，to字段填合约地址，nonce递增为1，data和value留空，签名广播后交易上链。

### 步骤7：验证执行结果

查合约0号存储槽：`cast storage 合约地址 0x`，返回`0x2a`（十进制42），合约逻辑执行成功。

* * *

## 八、关键避坑指南

1.  **nonce连续性问题**：交易必须按nonce从小到大执行，若前一笔nonce=2的交易未打包，nonce=3的交易永远不会被打包
    
2.  **交易覆盖的风险**：同nonce加速/取消交易，必须保证gasPrice比原交易高至少10%，否则很难被矿工优先打包，且成功率无法100%保证
    
3.  **gasLimit设置不当**：设置过低会导致交易执行到一半gas耗尽，状态回滚且gas不退；设置过高不会多扣费，未使用的gas会退还
    
4.  **合约部署的init代码误区**：init代码仅执行一次，永久上链的是init代码返回的运行时字节码，而非init代码本身
    
5.  **签名安全**：私钥绝对不能泄露，签名只能对自己组装的交易体执行，禁止对未知的交易哈希签名，防止资产被盗
    

* * *

## 核心速记清单

-   交易是**唯一能改链上状态**的载体，只能由EOA发起
    
-   7大字段决定交易的一切，**to字段决定交易类型**
    
-   部署合约to为空，data放init代码；调用合约to填合约地址，data放函数入参
    
-   交易执行完必出回执，**status=1成功，0失败**
    
-   EIP-2718让交易格式可扩展，兼容传统交易
    
-   nonce防重放，gas防资源滥用，签名防篡改
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->



## 一、EVM 核心定位与本质

### 1\. 基础定义

以太坊虚拟机（Ethereum Virtual Machine, EVM）是**以太坊“世界计算机”的核心**，是以太坊生态中负责交易执行、计算结果上链固化、全局状态更新的核心计算引擎。

### 2\. 核心本质

EVM 具备双重核心属性，是理解其设计的核心前提：

1.  **以太坊交易型状态机的状态转换函数**：以太坊整体可被定义为基于交易的状态机，而 EVM 就是这个状态机的核心转换规则，它决定了以太坊如何基于「当前世界状态」和「交易输入」，确定性地切换到「下一个世界状态」。
    
2.  **跨平台的栈式虚拟机**：EVM 是虚拟机范式的具体实现，通过平台无关的字节码实现了全节点执行环境的一致性，保证所有以太坊节点对交易执行结果达成共识，同时实现了硬件/操作系统的解耦。
    

## 二、以太坊的状态机模型

### 1\. 状态机通用理论

状态机是计算机科学中用于建模系统行为的抽象工具，核心由三个要素构成，文档以自动售货机为典型示例做了具象化解释：

| 状态机核心要素 | 定义 | 自动售货机示例 |
| 状态（S） | 系统在特定时间点的明确配置/条件，系统同一时间仅能处于一个状态 | 空闲（Idle）、等待选择（Selection）、出货中（Dispensing） |
| 输入（I） | 触发系统行为的动作、信号或环境变化，是状态切换的触发源 | 投币（InsertCoin）、选择商品（SelectDrink）、取货（CollectDrink） |
| 状态转换函数（Υ） | 定义系统基于「当前状态+输入」切换到下一个状态的规则，公式为 Υ(S,I) ⇒ S' | Υ(Idle,InsertCoin) ⇒ Selection；Υ(Idle,SelectDrink) ⇒ Idle（无效输入，状态不变） |

### 2\. 以太坊作为交易型状态机

以太坊完全符合状态机的定义，是一个**基于交易驱动的全局状态机**：

-   **世界状态**：以太坊的全局状态，本质是「20字节账户地址」到「账户状态」的映射，每个账户状态包含余额、nonce、存储、代码四大核心字段。
    
-   **输入**：以太坊交易（包括普通转账交易、合约调用交易、合约部署交易）。
    
-   **状态转换函数**：EVM 本身，所有交易的执行、状态的变更都必须通过 EVM 完成。
    
-   **状态更新规则**：实际运行中，多笔交易会被打包进一个区块，区块内所有交易执行完成后，生成的最终新状态会被追加到区块链上，形成链式的状态历史。
    

### 3\. 以太坊的两类账户

以太坊世界状态中仅存在两类账户，EVM 对其有明确的执行规则区分：

1.  **外部账户（EOA）**：由私钥控制的用户账户，无 EVM 代码，无法主动执行逻辑，仅能发起交易。
    
2.  **合约账户**：由非空 EVM 代码控制的账户，其代码就是俗称的「智能合约」；拥有独立的持久化存储，仅能在被交易/其他合约调用时，通过 EVM 执行代码逻辑。
    

## 三、虚拟机范式与 EVM 设计逻辑

### 1\. 传统软件的异构痛点

软件执行需要编译为目标硬件的指令集（ISA），而不同硬件（Intel/Apple Silicon）、不同操作系统的指令集不兼容，传统开发需要为每个平台单独编译原生二进制文件，开发和维护成本极高。

### 2\. 虚拟机的解决方案

虚拟机通过两层抽象解决了跨平台问题，也是 EVM 的核心设计思路：

1.  **第一层：平台无关的字节码抽象**：源码编译为统一的虚拟机字节码，字节码是由字节组成的指令序列，每个字节对应虚拟机的一个特定操作。
    
2.  **第二层：平台专属的虚拟机实现**：不同平台安装对应版本的虚拟机，由虚拟机将统一字节码翻译为当前平台的原生代码执行。
    

该方案带来两个核心优势：

-   **可移植性**：一次编译，全平台运行，无需重复适配；
    
-   **硬件抽象**：开发者无需关注底层硬件/操作系统细节，仅面向虚拟机开发。
    

典型案例：Java 的 JVM、Lua 的 LuaVM。

### 3\. EVM 的虚拟机核心设计

-   **字长**：EVM 的基础数据处理单位（字）为 32 字节（256 位），所有栈元素、存储槽位均基于该字长设计。
    
-   **核心目标**：为以太坊全节点提供**完全一致、确定性的执行环境**，保证同一笔交易在任何节点上执行，都能得到完全相同的状态结果，这是区块链共识的核心基础。
    
-   **实现载体**：EVM 的规范由以太坊客户端具体实现，主流实现包括 Go 语言的 Geth、Rust 语言的 revm、C++ 语言的 EVMONE、Python 语言的 py-evm 等。
    

## 四、EVM 字节码、操作码与汇编

### 1\. 字节码的本质

EVM 字节码是智能合约的最终执行形式，是由 8 位（1 字节）为单位组成的序列，其中每个字节分为两类：

-   **操作码（Opcode）**：1 字节长度，定义 EVM 要执行的具体操作（如加法、压栈、存储写入等）；
    
-   **操作数（Operand）**：操作码的输入参数，仅特定操作码附带，长度由操作码类型决定。
    

### 2\. 字节码的三种表示形式

| 表示形式 | 特点 | 示例 |
| 二进制 | EVM 最终执行的原生形式 | 01100000 00000110 01100000 00000111 00000001 |
| 十六进制 | 二进制的简写形式，日常开发最常用 | 60 06 60 07 01 |
| EVM 汇编 | 人类可读的最低级形式，为操作码提供助记符，是逆向、优化合约的核心工具 | PUSH1 06 PUSH1 07 ADD |

### 3\. 操作码核心规则

-   目前仅 `PUSH*` 系列操作码附带操作数，`PUSHX` 中的 X 定义了操作数的字节长度（如 `PUSH1` 附带 1 字节操作数，`PUSH32` 附带 32 字节操作数）。
    
-   操作码可通过以太坊改进提案（EIP）新增或修改，如 EIP-1153 新增了瞬态存储操作码 `TSTORE/TLOAD`。
    
-   完整的操作码列表可参考以太坊黄皮书附录 H，文档中核心常用操作码分类如下：
    

| 操作类别 | 核心操作码 | 功能说明 |
| 算术运算 | ADD、MUL | 栈顶两个值相加/相乘 |
| 栈操作 | PUSH1-PUSH32、DUP1-DUP16、SWAP1-SWAP16 | 压栈、复制栈元素、交换栈元素 |
| 内存操作 | MLOAD、MSTORE、MSTORE8、MSIZE | 内存的读写、大小查询 |
| 存储操作 | SLOAD、SSTORE | 合约持久化存储的读写 |
| 调用数据操作 | CALLDATALOAD、CALLDATACOPY | 交易入参的读取、复制 |
| 控制流操作 | JUMP、JUMPDEST、RETURN、STOP | 程序跳转、执行终止 |

### 4\. 程序计数器（PC）

-   **核心作用**：跟踪下一条待执行指令在字节码数组中的字节偏移量，保证 EVM 按顺序执行指令，或实现合法的跳转逻辑。
    
-   **执行规则**：默认情况下，每执行完一条指令，PC 按指令字节长度递增；遇到 `JUMP` 指令时，可直接将 PC 设置为栈顶的目标偏移值，实现分支、循环等动态控制流。
    
-   **安全规则**：`JUMP` 指令的目标位置必须是 `JUMPDEST` 操作码标记的位置，防止非法控制流跳转，保证执行安全。
    
-   **图灵完备性**：通过 `JUMP/JUMPDEST` 实现灵活的执行路径，让 EVM 具备图灵完备性；而 Gas 机制的存在，让 EVM 最终被定义为**准图灵完备**。
    

## 五、EVM 四大数据存储位置（核心重点）

EVM 执行过程中，数据分为四个核心存储区域，其生命周期、读写成本、访问规则、作用完全不同，是智能合约开发和优化的核心知识点。

### 1\. 栈（Stack）

-   **数据结构**：遵循后进先出（LIFO）原则，仅支持 PUSH（压入栈顶）、POP（弹出栈顶）两个核心操作。
    
-   **核心作用**：EVM 操作码的核心计算载体，是所有运算的“草稿纸”，负责存储计算中间值、为操作码提供入参，绝大多数操作码都基于栈完成执行。
    
-   **核心参数与限制**：
    

-   最大深度 1024 个元素，每个元素固定为 32 字节；
    
-   仅栈顶的 16 个元素可被直接访问，该限制由 `DUP/SWAP` 操作码的最大范围决定（DUP1-DUP16、SWAP1-SWAP16）；
    
-   空栈执行 POP 会触发栈下溢错误，栈深度超过 1024 会触发栈溢出错误，均会导致合约执行失败。
    

-   **生命周期**：单次合约调用执行结束后，栈会被完全重置，数据不保留。
    
-   **执行示例**：加法运算的栈执行流程
    

1.  执行 `PUSH1 06`：将 0x06 压入栈顶，栈：`[0x06]`
    
2.  执行 `PUSH1 07`：将 0x07 压入栈顶，栈：`[0x06, 0x07]`
    
3.  执行 `ADD`：弹出栈顶两个值相加，将结果 0x0d 压入栈顶，栈：`[0x0d]`
    

### 2\. 内存（Memory）

-   **本质**：线性字节数组，初始所有位置的值均为 0，理论最大长度为 2^256 字节，实际使用中动态扩容。
    
-   **核心作用**：补充栈的容量限制，存储单次合约执行内的临时数据，支持对任意大小数据的索引访问，用于处理复杂数据结构、函数入参/返回值等。
    
-   **核心操作**：
    

-   `MSTORE`：从栈顶取出偏移量和 32 字节值，将值写入内存指定偏移位置；
    
-   `MSTORE8`：与 `MSTORE` 逻辑一致，仅写入 1 个字节，而非完整 32 字节；
    
-   `MLOAD`：从栈顶取出内存偏移量，读取该偏移位置开始的 32 字节，压入栈顶；
    
-   `MSIZE`：将当前已扩容的内存总字节数压入栈顶。
    

-   **扩容规则**：内存按 32 字节（1 字）为单位的“页”动态分配，写入超出当前已扩容范围的位置时，会触发内存扩容，扩容会收取对应的 Gas 费用。
    
-   **关键特点**：无强制字节对齐要求，但 `MLOAD/MSTORE` 按 32 字节字操作；无单字节读取指令，需读取完整字后通过位掩码提取目标字节。
    
-   **生命周期**：单次合约调用执行结束后，内存会被完全清空，数据不保留。
    

### 3\. 调用数据（Calldata）

-   **本质**：交易或合约消息调用时，传递给 EVM 的只读输入数据，以字节序列的形式存储，是智能合约函数入参的载体。
    
-   **核心特点**：完全只读，执行过程中无法修改；Gas 成本远低于内存和存储，是优化合约 Gas 消耗的重要方向。
    
-   **核心操作**：
    

-   `CALLDATALOAD`：从栈顶取出偏移量，读取该偏移开始的 32 字节，压入栈顶；
    
-   `CALLDATACOPY`：将 calldata 中指定长度的片段，复制到内存的指定位置。
    

-   **生命周期**：仅在单次合约调用的生命周期内有效，调用结束后数据销毁。
    

### 4\. 存储（Storage）

-   **本质**：与合约账户绑定的字寻址字数组，是智能合约的持久化数据库，固定拥有 2^256 个槽位，每个槽位固定 32 字节。
    
-   **核心作用**：存储合约的状态变量，是以太坊世界状态的组成部分，合约的核心业务数据均存储于此。
    
-   **核心特点**：
    

-   **持久性**：数据跨交易、跨区块永久存储，写入后会固化到区块链上，除非合约代码主动修改，否则不会变更；
    
-   **访问权限**：仅所属合约的代码可进行读写操作，外部账户、其他合约均无法直接修改；
    
-   **底层实现**：合约账户的状态中包含一个存储根哈希，指向该合约独立的默克尔帕特里夏树（Merkle Patricia Trie），所有存储数据都保存在该树中。
    

-   **核心操作**：
    

-   `SSTORE`：从栈顶取出存储槽位编号和 32 字节值，将值写入该合约指定的存储槽位；
    
-   `SLOAD`：从栈顶取出存储槽位编号，读取该槽位的 32 字节值，压入栈顶。
    

-   **Gas 特性**：
    

-   读写成本是四类存储中最高的，因为存储数据需要全以太坊节点同步、永久保存，占用全网状态资源；
    
-   当 `SSTORE` 操作将一个槽位的值从非零改为零时，可获得 Gas 退款，退款会计入交易的退款计数器，在交易执行结束后抵扣部分 Gas 成本，用于激励开发者释放区块链状态资源。
    

-   **优化技巧**：高级语言（如 Solidity）会自动将多个总尺寸小于等于 32 字节的小变量，打包到同一个存储槽位中，减少存储操作次数，降低 Gas 消耗。
    

## 六、Gas 机制

### 1\. 设计初衷

EVM 中无限循环等恶意代码会消耗全网节点的计算资源，引发 DoS 攻击；Gas 机制通过为计算资源定价，限制了合约执行的最大计算步骤，从根本上解决了该问题。

### 2\. 核心定义

-   Gas 是 EVM 中计算资源的计价单位，每一个操作码都对应固定的 Gas 成本，操作的计算/存储复杂度越高，Gas 成本越高（具体成本参考以太坊黄皮书附录 G）。
    
-   交易发起者需要设置 `gasPrice`（单位 Gas 的价格，以 ETH 计价）和 `gasLimit`（本次交易允许消耗的最大 Gas 量），并提前预付 Gas 费用。
    

### 3\. 执行规则

1.  交易执行过程中，EVM 会逐操作码扣减对应 Gas，若执行过程中 Gas 耗尽，会立即终止执行，所有状态变更回滚，已消耗的 Gas 不予退还。
    
2.  若交易正常执行完成，剩余未消耗的 Gas 会退还给交易发起者。
    

### 4\. 准图灵完备性

EVM 具备图灵完备的执行能力，可实现任意复杂的逻辑；但由于 Gas 机制的存在，任何执行都被限制在有限的计算步骤内，无法执行无限循环，因此 EVM 被定义为**准图灵完备**。

## 七、EVM 完整执行流程

1.  **输入接收**：EVM 接收执行输入，包括交易/合约调用的 calldata、当前以太坊世界状态、交易上下文（gasLimit、gasPrice、发送方地址、转账金额、签名等）。
    
2.  **执行环境初始化**：初始化空的栈、内存，将程序计数器 PC 置 0，将可用 Gas 设置为交易的 gasLimit。
    
3.  **指令循环执行**：
    

-   PC 读取当前字节码位置的指令，解析为操作码和对应的操作数；
    
-   扣减该操作码对应的 Gas，若可用 Gas 不足，立即终止执行，触发 out-of-gas 异常；
    
-   执行操作码对应的逻辑，完成对栈、内存、存储、calldata 的对应操作；
    
-   更新 PC 值，默认按指令长度递增，跳转指令则直接设置为目标偏移量。
    

4.  **执行终止**：遇到 `RETURN`、`STOP`、`REVERT` 指令，或触发 Gas 耗尽、栈异常、非法跳转等执行错误时，终止执行。
    
5.  **状态提交**：若正常执行终止，将本次执行产生的状态变更写入以太坊世界状态；若执行异常，所有状态变更回滚，已消耗的 Gas 不予退还。
    

## 八、EVM 的升级与演进

### 1\. 升级原则

EVM 升级以**向后兼容**为核心原则，避免重大变更破坏已部署的合约和上层开发语言；仅会进行不破坏原有逻辑的修改，如新增操作码、调整现有操作码的 Gas 成本、优化执行模型等。

### 2\. 典型升级案例

-   EIP-1153：新增瞬态存储操作码 `TSTORE/TLOAD`，用于交易内的临时数据存储，降低跨函数调用的 Gas 成本；
    
-   EIP-6780：弱化 `SELFDESTRUCT` 操作码的功能，仅在合约部署的同一交易内生效，避免破坏现有合约的兼容性；
    
-   EOF（Ethereum Object Format）：EVM 的重大升级，定义了标准化的字节码格式，让 EVM 可以更高效地解析和执行字节码，包含多个关联 EIP，经过了长期的社区讨论和打磨。
    

## 九、开发与学习补充

1.  **开发层级**：日常开发中，开发者极少直接编写 EVM 汇编，仅在极致的 Gas 优化、合约逆向分析时使用；绝大多数场景下，开发者使用 Solidity、Vyper、Huff、Fe 等高级语言编写智能合约，再由编译器将源码编译为 EVM 字节码。
    
2.  **核心学习与调试工具**：
    

-   [evm.codes](http://evm.codes)：操作码参考手册 + 交互式执行 playground，是学习 EVM 字节码的核心工具；
    
-   [evm.storage](http://evm.storage)：合约存储可视化浏览器，用于分析合约存储布局；
    
-   EVM puzzles：EVM 逆向解谜练习，用于提升字节码理解能力；
    
-   [ETH.BUILD](http://ETH.BUILD)：可视化的 EVM 基础教学工具。
    

3.  **权威学习资源**：
    

-   以太坊黄皮书（Gavin Wood 著）：EVM 规范的官方定义文档；
    
-   《Mastering Ethereum》：以太坊技术详解权威著作；
    
-   《Ethereum Book》第 13 章：EVM 专项详解。
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->




# 以太坊执行层（EL）架构 核心笔记

## 一、执行层核心定位与整体架构

### 1\. 核心职责（合并后The Merge）

以太坊合并后，原共识相关职责完全移交共识层（CL），执行层（EL）客户端核心职能聚焦为：

-   区块链数据合法性校验，维护链数据本地副本
    
-   基于DevP2P协议实现P2P网络通信，完成节点间区块、交易的 gossip 传播
    
-   维护交易池（mempool），校验交易合法性并全网广播
    
-   响应共识层指令，执行交易、完成以太坊全局状态转换
    
-   对外提供标准化JSON-RPC API，对内通过专属Engine API与共识层通信
    

### 2\. 整体架构层级

**驱动链路**：共识层（CL）→ Engine API → 执行层核心（执行引擎）→ DevP2P网络层

-   执行引擎是EL的核心，完全由CL通过Engine API驱动
    
-   DevP2P为EL提供网络接入能力，通过合法启动节点完成网络初始化
    
-   架构设计遵循**职责分离原则**：CL负责共识与分叉选择，EL负责交易验证与执行
    

## 二、EL核心基础组件

### 1\. EVM（以太坊虚拟机）

-   核心定位：确定性虚拟执行引擎，解决底层硬件异构导致的执行结果不一致问题，保障全节点计算结果达成共识
    
-   设计理念：**三明治复杂度模型**——外层（EVM字节码）保持简洁，复杂度集中在中间层（Solidity高级语言→EVM字节码的编译器）
    
-   核心作用：执行智能合约逻辑，是触发以太坊状态转换的执行载体
    

### 2\. State（全局状态）

-   核心定位：以太坊是基于全局状态的状态机，这是与比特币UTXO模型的核心差异
    
-   状态内容：账户地址与余额、合约代码与存储数据、当前链状态与网络状态
    
-   底层结构：通过Merkle-Patricia Tries（MPT）存储，保障状态的可验证性与防篡改性
    

### 3\. Transactions（交易）

-   唯一作用：作为输入触发以太坊的状态转换
    
-   核心流程：交易进入mempool→节点校验合法性→EVM内执行→校验通过则完成全局状态变更
    
-   关键规则：区块内任意一笔交易执行失败，整个区块将被判定为无效
    

### 4\. DevP2P网络层

-   核心定位：EL客户端之间P2P通信的标准接口层
    
-   核心功能：实现交易与区块的全网传播；节点间链数据同步；接收数据后先校验合法性再广播，防范垃圾交易与无效状态转换
    

### 5\. JSON-RPC API

-   核心定位：外部应用（钱包、DApp）与EL交互的标准化接口
    
-   核心能力：支持外部查询以太坊链上状态、发送钱包签名后的交易；交易由EL完成校验后，通过DevP2P全网广播
    

## 三、Engine API：CL与EL通信的核心

### 1\. 基础规则

-   专属权限：仅用于CL与EL的内部通信，基于带JWT认证的JSON-RPC over HTTP，不对外暴露
    
-   认证说明：JWT仅用于校验发送方为合法CL客户端，不加密通信流量
    
-   版本兼容：支持V1/V2/V3三个版本，节点启动时先完成能力协商
    

### 2\. 前置流程：能力交换

-   方法：`engine_exchangeCapabilities`
    
-   执行时机：节点启动时，常规业务运行前
    
-   核心作用：CL与EL互相交换支持的Engine API方法与版本，协商通用协议版本，保障兼容性，兼顾新特性落地与向后兼容
    

### 3\. 核心端点与功能

（1）New Payload (V1/V2/V3)

-   核心作用：执行负载（Execution Payload）的验证与链上插入
    
-   触发时机：CL收到网络中新的信标区块，提取其中的执行负载后调用
    
-   校验流程：
    

1.  校验负载头中的父区块哈希存在，且与本地链的预期父块匹配
    
2.  验证额外执行承诺（如Cancun升级后的相关数据）
    
3.  在EVM中执行负载内的所有交易，完成状态更新
    

-   核心返回状态：
    

-   `VALID`：完全执行且所有校验通过
    
-   `INVALID`：负载或其祖先区块校验失败
    
-   `SYNCING`：EL仍在链同步中，缺失相关区块
    
-   `ACCEPTED`：基础校验通过，完整执行待完成（浅状态客户端常见）
    

（2）Fork Choice Updated (V1/V2/V3)

-   核心作用：管理EL链状态同步，触发区块构建
    
-   触发时机：CL按LMD-GHOST算法更新分叉选择规则，或验证者被分配出块权限时
    
-   执行流程：
    

1.  EL更新本地规范链头（canonical head）
    
2.  若CL传入负载属性，EL立即启动区块构建流程
    
3.  返回处理状态，若启动区块构建则同步返回`payloadId`
    

-   返回状态：与New Payload一致，`VALID`表示分叉选择更新处理成功，EL链状态已同步至最新
    

（3）辅助端点：`engine_getPayload`

-   核心作用：CL通过Fork Choice Updated返回的`payloadId`，从EL获取已构建完成的执行负载，用于信标区块打包
    

## 四、核心工作流程

### 1\. 节点启动全流程

1.  CL调用`engine_exchangeCapabilities`，与EL协商支持的Engine API版本
    
2.  CL发送初始`engine_forkchoiceUpdated`调用（无负载属性），告知EL当前链的分叉选择状态
    
3.  EL若未完成链同步，返回`SYNCING`状态；同步完成后返回`VALID`状态，节点进入正常运行模式
    

### 2\. 验证者常规运行流程（每个Slot）

1.  CL调用`engine_forkchoiceUpdated`，更新EL的本地链状态
    
2.  出块场景：若该验证者被分配本轮出块权限，CL在调用中传入负载属性，触发EL构建区块；EL返回`payloadId`，CL后续通过`engine_getPayload`获取完整执行负载
    
3.  区块校验场景：若收到其他验证者提出的信标区块，CL提取执行负载，调用`engine_newPayload`，由EL完成负载全量校验
    

### 3\. 核心执行逻辑：状态转换函数（STF）

STF是EL的核心功能，是`New Payload`的底层实现，负责完成区块校验与状态更新。

-   入参：父区块、当前待校验区块、父区块对应的状态数据库（StateDB）
    
-   返回值：交易执行后的新状态DB；执行失败则返回错误，不更新原有状态
    
-   执行步骤：
    

1.  **区块头校验**：验证父哈希合法性、区块号连续性、gas limit合规性（单块调整幅度不超1/1024）、EIP-1559基础费更新正确性等
    
2.  **交易逐笔执行**：在EVM中按顺序执行区块内每笔交易，传入区块头上下文、交易数据、当前状态
    
3.  **状态提交**：所有交易执行成功后，更新状态DB，生成新的状态根
    

## 五、节点同步机制

### 1\. 同步基础规则

-   触发源：由CL的LMD-GHOST分叉选择规则驱动，通过Engine API的Fork Choice Updated端点通知EL执行同步
    
-   核心动作：从对等节点下载远程区块，在EVM中完成区块全量校验
    
-   同步状态：通过`VALID/INVALID/SYNCING/ACCEPTED`标识EL当前的同步进度
    

### 2\. 两种核心同步策略对比

| 特性 | Full Sync（全量同步） | Snap Sync（快照同步） |
| 核心逻辑 | 从创世块开始，逐块重放所有历史交易，一步步重建状态树 | 以最近终局块为锚点，直接下载状态树叶子节点+默克尔证明，本地重建完整状态 |
| 核心流程 | 1. 下载创世到链头的所有区块头/体2. 逐块在EVM中执行所有交易3. 校验本地状态根与链头状态根匹配 | 1. 选定终局锚点块，获取其状态根2. 下载状态树叶子（账户、存储槽）、合约字节码3. 批量校验默克尔证明，写入快照DB4. 修复阶段：补全缺失数据，保障状态完整一致5. 基于锚点状态，执行后续区块至链头 |
| 安全性 | 最高，全量校验所有历史状态转换 | 锚点块基于弱主观性校验，后续区块全量校验 |
| 性能 | 主网同步需数天，占用极高CPU、磁盘、网络资源 | 主网同步缩短至数小时，仅修复阶段硬件压力较大 |
| 兼容性 | EIP-4444全面实施后，将不再支持从创世块的全量同步 | EIP-4444后成为主流同步方案，基于检查点同步 |

## 六、EL其他核心组件

### 1\. 交易池（Transaction Pools）

核心作用：存储待打包的合法交易，为区块构建提供交易来源，通过DevP2P实现全网交易传播。分为两大类型：

-   **传统交易池（Legacy Pools）**：基于价格排序的堆/优先级队列，按有效小费、gas费上限双维度排序，池饱和时按堆大小驱逐低优先级交易
    
-   **Blob交易池（Blob Pools）**：为EIP-4844 Blob交易设计，基于对数函数的驱逐队列，通过优先级堆管理交易驱逐逻辑
    

### 2\. 内置共识引擎（Ethone）

-   定位：EL自带的轻量共识引擎，仅保留完整共识引擎约一半的功能，兼容PoS、PoA（Clique）、PoW（Ethash）三种共识机制
    
-   核心作用：处理合并前（TTD前）的历史区块共识校验，兼容不同共识机制的区块处理规则
    
-   合并后规则：PoS合并后的区块，难度固定为0、叔块数必须为0、nonce固定为0，相关共识逻辑全部移交CL处理
    

### 3\. 存储后端

EL需要持久化两类核心数据：区块链历史数据（远古数据库）、当前状态与近期状态（MPT结构），主流存储后端如下：

-   **LevelDB**：早期主流方案，基于LSM树的嵌入式KV数据库，现已因停止维护被多数客户端弃用
    
-   **Pebble**：LevelDB替代方案，Geth当前默认后端，优化LSM树架构，支持多活跃内存表，降低写停顿，更适配以太坊写密集、低延迟的业务需求
    
-   **MDBX**：其他EL客户端广泛采用的高性能存储方案，优化读写性能与数据一致性
    

## 七、主流EL客户端：Geth核心实现

Geth是以太坊最主流的执行层客户端，其核心模块与EL架构完全对齐：

1.  **交易执行**：交易先进入mempool，完成签名、nonce、gas费校验后，被打包进区块，在EVM中执行后更新账户余额、合约存储等状态
    
2.  **区块处理**：按顺序执行区块内所有交易，执行完成后提交最终状态，存储状态根哈希保障链一致性
    
3.  **网络通信**：基于DevP2P协议实现节点间区块、交易的P2P传播，接收数据先完成合法性校验再转发
    
4.  **EVM执行**：内置标准EVM，处理所有智能合约逻辑，保障执行的确定性
    
5.  **状态同步**：原生支持Full Sync与Snap Sync两种同步模式
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->





# 以太坊执行层规范 (EL Specs) 学习笔记

> 基于 [epf.wiki/#/wiki/EL/el-specs](https://epf.wiki/#/wiki/EL/el-specs) 整理  
> 参考来源：EELS 官方博客、ethereum/execution-specs、OpenZeppelin 技术解析

* * *

## 目录

1.  [规范概览](#1-%E8%A7%84%E8%8C%83%E6%A6%82%E8%A7%88)
    
2.  [规范文档体系](#2-%E8%A7%84%E8%8C%83%E6%96%87%E6%A1%A3%E4%BD%93%E7%B3%BB)
    
3.  [EELS：可执行规范](#3-eels%E5%8F%AF%E6%89%A7%E8%A1%8C%E8%A7%84%E8%8C%83)
    
4.  [EELS 架构与模块结构](#4-eels-%E6%9E%B6%E6%9E%84%E4%B8%8E%E6%A8%A1%E5%9D%97%E7%BB%93%E6%9E%84)
    
5.  [核心功能深入：自下而上](#5-%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%E6%B7%B1%E5%85%A5%E8%87%AA%E4%B8%8B%E8%80%8C%E4%B8%8A)
    
6.  [EIP 流程与规范演进](#6-eip-%E6%B5%81%E7%A8%8B%E4%B8%8E%E8%A7%84%E8%8C%83%E6%BC%94%E8%BF%9B)
    
7.  [关键 API 规范](#7-%E5%85%B3%E9%94%AE-api-%E8%A7%84%E8%8C%83)
    
8.  [测试框架](#8-%E6%B5%8B%E8%AF%95%E6%A1%86%E6%9E%B6)
    
9.  [重要概念速查](#9-%E9%87%8D%E8%A6%81%E6%A6%82%E5%BF%B5%E9%80%9F%E6%9F%A5)
    

* * *

## 1\. 规范概览

以太坊执行层（Execution Layer，EL）的规范并非一份单一文档，而是由多个相互配合的规范共同构成：

| 规范 | 作用 | 仓库 |
| --- | --- | --- |
| EELS（Python 可执行规范） | 核心协议参考实现，每个 Fork 快照 | ethereum/execution-specs |
| 黄皮书（Yellow Paper） | 原始数学形式化规范（已逐步被 EELS 取代） | ethereum/yellowpaper |
| EIP（以太坊改进提案） | 描述协议变更的标准，仅记录 diff | ethereum/EIPs |
| Execution APIs（JSON-RPC） | 执行客户端对外暴露的接口规范 | ethereum/execution-apis |
| Engine API | EL 与 CL 之间的内部通信接口 | ethereum/execution-apis/engine |
| execution-spec-tests | 用于验证客户端合规性的测试框架 | ethereum/execution-spec-tests |

* * *

## 2\. 规范文档体系

### 2.1 历史演进：从黄皮书到 EELS

```
黄皮书（2014）
  └── 数学符号表达，密集难读
      ↓ 逐渐难以维护（post-Merge 后未同步更新）
EELS（2021~2023 公开）
  └── Python 实现，可读性优先
  └── 每个 Fork 独立快照
  └── 支持 EIP 原型验证
```

-   **黄皮书的局限**：使用晦涩的数学符号（如 $\\sigma$、$\\mu$ 等），The Merge 后逐渐不再与主网同步，对新贡献者极不友好
    
-   **EELS 的目标**：成为"精神上的黄皮书继承者"，面向程序员，可直接运行和测试
    

### 2.2 EIP 与 EELS 的关系

```
EIP（描述变更）  →  EELS（实现变更）  →  生产客户端（Geth/Nethermind/...）
       ↓
  只记录 diff                完整协议快照              混合多个 Fork 在同一代码库
```

**EIP 的不足**：只记录修改内容，无法展示协议全貌；而 EELS 为每个 Fork 提供完整的协议快照，大幅降低理解难度。

* * *

## 3\. EELS：可执行规范

### 3.1 什么是 EELS

**EELS**（Ethereum Execution Layer Specification）是以太坊执行层的 **Python 参考实现**，核心特性：

-   🎯 **可读性优先**：代码风格偏向表达协议意图，而非追求运行性能
    
-   📸 **Fork 快照**：为每个硬分叉提供完整的独立代码快照，并提供 Fork 间的 diff 视图
    
-   🧪 **可执行**：能够填充和运行状态测试，跟踪主网状态
    
-   🔧 **EIP 原型平台**：EIP 作者可在 EELS 中快速原型化其提案
    

### 3.2 EELS 的局限

EELS **不实现**以下功能：

-   P2P 网络（需依赖生产客户端同步区块）
    
-   JSON-RPC API（由 `ethereum/execution-apis` 单独维护）
    
-   高性能优化（仅为参考，不用于生产环境）
    

### 3.3 EELS 的价值

> "EELS 是 EIP 作者进行原型设计的首选，也是理解以太坊工作方式的最佳参考。"

对不同受众的意义：

-   **智能合约开发者**：快速查阅特定 EVM 指令在某 Fork 的精确行为
    
-   **客户端开发者**：查看 Fork 间的精确 diff，理解需实现的变更
    
-   **研究者/EIP 作者**：在正式规范之前先在 EELS 中验证方案可行性
    

* * *

## 4\. EELS 架构与模块结构

### 4.1 两大模块分类

```
EELS 快照
├── EVM 模块（vm/）
│   ├── 操作码（opcodes）实现
│   ├── 预编译合约（precompiles）
│   ├── Gas 计算
│   ├── 栈、内存管理
│   └── 解释器（interpreter）：执行 EVM 消息的入口
│
└── 区块链执行模块（blockchain execution）
    ├── 区块（blocks）与交易（transactions）数据结构
    ├── 状态（state）与状态树（trie）
    ├── 区块验证与处理逻辑
    └── Fork 管理（fork.py）
```

### 4.2 Prague Fork 文件结构（当前主网）

```
src/ethereum/prague/
├── blocks.py          # 区块结构定义
├── bloom.py           # Bloom 过滤器
├── exceptions.py      # 异常类型
├── fork_types.py      # Fork 相关类型
├── fork.py            # 主入口：区块处理、状态转换
├── requests.py        # EL→CL 请求（EIP-7685）
├── state.py           # 世界状态管理
├── transactions.py    # 交易类型定义
├── trie.py            # Merkle Patricia Trie
├── utils/             # 工具函数
└── vm/
    ├── eoa_delegation.py     # EIP-7702 EOA 委托
    ├── exceptions.py
    ├── gas.py                # Gas 计算
    ├── instructions/         # 全部操作码实现
    ├── interpreter.py        # EVM 解释器（核心入口）
    ├── memory.py             # 内存管理
    ├── precompiled_contracts/ # 预编译合约
    ├── runtime.py
    └── stack.py              # 栈操作
```

> **注意**：不同 Fork 的文件结构可能有所不同——某些 EIP 会新增或移除特定文件。

* * *

## 5\. 核心功能深入：自下而上

以下按从底层到顶层的顺序梳理执行层的核心流程：

```
EVM 消息（Message）
    ↓
用户交易（User Transactions）
    ↓
系统交易（System Transactions）
    ↓
区块处理（Block Processing）
    ↓
状态转换（State Transition）
```

### 5.1 EVM 消息（Message）

**入口函数**：`process_message_call(message: Message) -> MessageCallOutput`

```python
def process_message_call(message: Message) -> MessageCallOutput
```

`Message` 的关键字段：

| 字段 | 说明 |
| --- | --- |
| caller | 消息发送者（即 Solidity 的 msg.sender） |
| target | 目标地址（空 = 合约创建） |
| current_target | 实际执行目标（创建时与 target 不同） |
| code | 待执行的字节码 |
| gas | Gas 限额 |
| value | 转移的 ETH 数量（wei） |

**执行分支**：

-   `target` 为空 → **合约创建**：执行 init code，设置新合约 nonce=1，生成 runtime code
    
-   `target` 有地址 → **消息调用**：转移 value，循环执行字节码，一次一条 opcode
    

**返回值** `MessageCallOutput`：

-   剩余 Gas、Gas 退款、事件日志、返回数据
    

### 5.2 系统交易（System Transactions）

> **引入时间**：Cancun Fork（2024），始于 EIP-4788

系统交易的特殊性：

-   由节点**自动创建并执行**，不来自外部用户
    
-   `tx.origin` 和 `msg.sender` 均为 `SYSTEM_ADDRESS = 0xfff...ffe`
    
-   目标是**系统合约**（System Contract）：有状态的普通合约，但系统地址有特殊写权限
    
-   **不计入区块 Gas 限额**，不执行 EIP-1559 销毁语义
    

**两种变体**：

| 变体 | 说明 | 使用场景 |
| --- | --- | --- |
| checked（检查型） | 若目标无代码或执行失败则区块无效 | EIP-7002（提款请求）、EIP-7251（合并请求） |
| unchecked（非检查型） | 不执行任何检查 | EIP-4788（Beacon 根）、EIP-2935（历史存储） |

**当前系统合约**（Prague Fork）：

-   `BeaconRoots`（EIP-4788）：存储最近 8191 个 Beacon 区块根，供 EVM 查询
    
-   `HistoryStorage`（EIP-2935）：存储区块哈希历史
    
-   `WithdrawalRequest`（EIP-7002）：允许验证者从 EL 侧触发提款
    
-   `ConsolidationRequest`（EIP-7251）：允许验证者合并质押余额
    

### 5.3 用户交易（User Transactions）

用户交易由外部账户签名后广播到网络。目前支持的交易类型：

| 类型 ID | 名称 | 说明 |
| --- | --- | --- |
| Legacy | LegacyTransaction | 原始格式（无类型前缀） |
| 0x01 | AccessListTransaction | EIP-2930，支持访问列表 |
| 0x02 | FeeMarketTransaction | EIP-1559，引入 baseFee + tip |
| 0x03 | BlobTransaction | EIP-4844，携带 blob 数据 |
| 0x04 | SetCodeTransaction | EIP-7702，EOA 临时委托智能合约代码 |

**交易处理流程**：

1.  验证签名与 nonce
    
2.  预扣 Gas（`gas_limit * max_fee_per_gas`）
    
3.  执行消息调用（`process_message_call`）
    
4.  退还未使用的 Gas
    
5.  支付优先费（tip）给区块提议者
    
6.  销毁 base fee（EIP-1559）
    

### 5.4 区块处理（Block Processing）

区块处理在 `fork.py` 中的 `apply_body` 函数实现：

```
apply_body(block_env, block_body):
  1. 执行系统交易（BeaconRoots、HistoryStorage 等）
  2. 逐笔处理用户交易
  3. 处理提款（withdrawals）
  4. 处理 EL→CL 请求（requests）
  5. 验证 Gas 用量、收据根、日志 Bloom 等
  6. 更新状态根
```

### 5.5 状态转换（State Transition）

**顶层函数**：`state_transition(chain: BlockChain, block: Block) -> None`

```python
def state_transition(chain: BlockChain, block: Block) -> None:
    # 1. 验证区块头（时间戳、Gas 上限、父区块哈希等）
    # 2. 调用 apply_body 执行区块内容
    # 3. 验证最终状态根
    # 4. 将区块追加到链
```

这是执行层的最高层接口，接受来自共识层（通过 Engine API）的新区块，执行并更新链状态。

* * *

## 6\. EIP 流程与规范演进

### 6.1 EIP 的生命周期

```
Idea（想法）
  → Draft（草稿）：在 ethereum/EIPs 提 PR
  → Review（审核）：技术讨论、安全分析
  → Last Call（最后征询）：公开意见期
  → Final（最终）：被接受，等待 Fork 纳入
  → 纳入 Fork：在 EELS 中实现 → 编写测试 → 客户端实现
```

### 6.2 EIP 的分类

| 类别 | 说明 | 例子 |
| --- | --- | --- |
| Core EIP | 协议核心变更，需要硬分叉 | EIP-1559、EIP-4844 |
| ERC | 应用层标准（Token 标准等） | ERC-20、ERC-721 |
| Networking EIP | P2P 网络相关 | EIP-2364 |
| Interface EIP | ABI、JSON-RPC 等接口 | EIP-1474 |
| Meta EIP | 流程、治理相关 | EIP-1 |

### 6.3 Fork 激活机制演进

| 时期 | 激活方式 | 说明 |
| --- | --- | --- |
| Frontier ~ Bellatrix | 区块高度（block number） | 到达指定区块号后激活 |
| Paris（The Merge） | 总难度（Total Difficulty） | TD 到达 58750000000000000000000 |
| Shanghai 之后 | 时间戳（timestamp） | 更精确，适配 PoS slot 机制 |

* * *

## 7\. 关键 API 规范

### 7.1 JSON-RPC API

执行客户端对外暴露的标准接口，规范位于 `ethereum/execution-apis`：

-   所有执行客户端必须实现的统一接口
    
-   使用 OpenRPC 规范编写（YAML 格式）
    
-   主要命名空间：`eth_`、`net_`、`web3_`
    

常用方法示例：

```
eth_getBlockByHash      # 按哈希获取区块
eth_getTransactionByHash # 获取交易详情
eth_call                # 执行只读调用
eth_estimateGas         # 估算 Gas 用量
eth_sendRawTransaction  # 广播已签名交易
```

### 7.2 Engine API

EL（执行层）与 CL（共识层）之间的**内部通信接口**，是 The Merge 引入的关键设计：

| 方法 | 方向 | 说明 |
| --- | --- | --- |
| engine_forkchoiceUpdated | CL → EL | 通知 EL 当前最优链头（head/safe/finalized） |
| engine_newPayload | CL → EL | 提交新执行载荷（一个区块的交易列表）请求验证 |
| engine_getPayload | CL → EL | 请求 EL 构建新区块（出块时） |
| engine_exchangeCapabilities | 双向 | 协商双方支持的 Engine API 版本 |

**典型流程**：

```
CL 收到新 Beacon Block
  → engine_newPayload（发送执行载荷给 EL 验证）
  → EL 返回 VALID / INVALID / SYNCING
  → engine_forkchoiceUpdated（更新链头）
```

* * *

## 8\. 测试框架

### 8.1 execution-spec-tests

`ethereum/execution-spec-tests` 是执行层规范的**合规性测试框架**：

-   测试用例用 **Python 编写**（而非手动编写 JSON fixtures）
    
-   通过 `t8n` 工具生成 JSON fixtures，可被任意执行客户端消费
    
-   主要关注**近期和即将到来的**规范变更
    
-   与 `ethereum/tests`（传统测试套件）互补，不互相取代
    

### 8.2 测试流程

```
Python 测试代码 (tests/**/*.py)
  ↓ 由 t8n 工具执行
JSON Fixtures（状态测试、区块测试）
  ↓ 被各执行客户端消费
验证客户端实现是否符合 EELS 规范
```

支持的 `t8n` 工具：

-   `evm t8n`（go-ethereum/Geth）
    
-   其他执行客户端提供的 t8n 工具
    

* * *

## 9\. 重要概念速查

| 概念 | 解释 |
| --- | --- |
| EELS | Ethereum Execution Layer Specification，Python 参考实现 |
| Fork 快照 | EELS 为每个硬分叉维护一份完整的独立代码，互不干扰 |
| 系统交易 | 节点自动创建执行的交易，用于协议级系统合约调用 |
| 系统合约 | 由系统地址（0xfff...ffe）管理的特殊链上合约 |
| EIP-7702 | 允许 EOA 临时委托智能合约代码（账户抽象的关键一步） |
| EIP-4844 Blob | 携带 blob 数据的新交易类型，用于降低 L2 数据成本 |
| Engine API | EL 与 CL 之间的内部 JSON-RPC 接口，The Merge 后引入 |
| t8n 工具 | 状态转换工具，用于生成测试 fixtures |
| EL→CL 请求 | EIP-7685 引入的跨层请求机制（如 EIP-7002 提款请求） |
| Precompile | 预编译合约，用原生代码（而非 EVM 字节码）实现的内置功能 |

* * *

## 延伸资源

-   📦 **EELS 仓库**：[github.com/ethereum/execution-specs](https://github.com/ethereum/execution-specs)
    
-   🌐 **EELS 渲染文档**：[ethereum.github.io/execution-specs](https://ethereum.github.io/execution-specs)
    
-   🔍 **Fork 间 Diff**：[ethereum.github.io/execution-specs/diffs](https://ethereum.github.io/execution-specs/diffs/index.html)
    
-   📋 **Execution APIs**：[github.com/ethereum/execution-apis](https://github.com/ethereum/execution-apis)
    
-   🧪 **Spec Tests**：[github.com/ethereum/execution-spec-tests](https://github.com/ethereum/execution-spec-tests)
    
-   📰 **EELS 官方博客**：[blog.ethereum.org/2023/08/29/eel-spec](https://blog.ethereum.org/2023/08/29/eel-spec)
    

* * *

_整理时间：2026年4月 | 基于 epf.wiki EL/el-specs 页面内容_
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->






# **Ethereum Protocol**

# 一、前史：密码朋克与比特币 (Prehistory)

**以太坊的诞生不是凭空而来，而是几十年密码学、隐私运动与数字货币实验积累的产物。**

**1.1 密码朋克运动**

**1980年代，David Chaum 开创了匿名数字现金与假名声誉系统的研究。1992年，Eric Hughes、Timothy C. May 和 John Gilmore 在旧金山湾区创立了密码朋克（Cypherpunks）小组，核心信念是：**

-   隐私是开放社会不可或缺的权利
    
-   强加密技术是个人自由的工具，不应受到政府管控
    
-   去中心化系统是对抗审查与监控的解药
    

| 核心宣言"Privacy is necessary for an open society in the electronic age." — Eric Hughes, A Cypherpunk's Manifesto (1993) |

**1.2 数字货币的先驱技术**

**在比特币出现前，密码朋克们已做出一系列关键技术尝试：**

| 年份 | 项目/人物 | 贡献 |
| 1991 | Hashcash (Adam Back) | 工作量证明（PoW）的前身，用于反垃圾邮件，后被比特币直接采用 |
| 1998 | B-money (Wei Dai) | 提出匿名、分布式电子现金系统，引入 PoW 作为铸币机制，在密码朋克邮件列表中发布 |
| 1998 | Bit Gold (Nick Szabo) | 设计去中心化数字货币，结合了智能合约思想与 PoW 共识算法，但未能解决双花问题 |
| 2004 | RPOW (Hal Finney) | 可重用工作量证明，首次实现类似UTXO 的一次性密码代币，但验证仍依赖中心服务器 |

  
  

**1.3 比特币的诞生与局限**

**2008年，Satoshi Nakamoto 发布比特币白皮书，2009年创世区块上线。比特币综合了 PoW、P2P 网络和密码学，第一次真正解决了去中心化双花问题。但比特币被设计为"极简计算器"——只支持货币转账，脚本功能非常有限，无法支撑复杂的去中心化应用。**  
  

**Vitalik Buterin 在 2011 年接触比特币，意识到需要一个更通用的平台——不仅仅是货币，而是一台"全球计算机"，能运行任意程序（智能合约）。这正是以太坊诞生的思想根源。**

# 二、以太坊协议架构 (Architecture)

**以太坊是一个分层的、双客户端架构系统。The Merge（2022年）之后，以太坊明确分为执行层（EL）和共识层（CL），通过 Engine API 协同工作。**  
  

## 2.1 三层嵌套模型

| 核心概念物理节点（Node）→ 运行以太坊客户端软件的真实计算机以太坊网络（Network）→ 全球所有节点组成的 P2P 网络以太坊虚拟机（EVM）→ 安装在节点上的可信计算平台，执行智能合约 |

**每个节点都是完整副本，这与客户端-服务器模型根本不同。所有参与以太坊的计算机都拥有完整的区块链状态，共同维护网络的去中心化与安全。**

## 2.2 执行层（Execution Layer）

**执行层负责处理交易和运行智能合约，是"计算"发生的地方。**

-   **核心：**EVM（以太坊虚拟机）
    

-   解释和执行智能合约字节码
    
-   维护账户余额与合约状态
    
-   通过 **Gas 机制** 防止滥用
    

-   **账户：**账户类型：EOA（外部账户，用户私钥控制）和 CA（合约账户，由代码控制）
    
-   **状态转换：**state\_transition 函数：接收新区块 → 验证有效性 → 更新状态树
    
-   **数据结构：**交易 → 区块 → 状态根（Merkle Patricia Trie）
    

主流执行客户端：Geth（Go）、Nethermind（C#）、Besu（Java）、Erigon（Go/重构）、Reth（Rust）

## 2.3 共识层（Consensus Layer / Beacon Chain）

**共识层负责验证者管理、区块最终确定性，是"谁有权写链"的决策机构。**

-   验证者需质押 32 ETH 参与出块与投票
    
-   使用 Gasper 协议（Casper FFG + LMD-GHOST）实现最终确定性
    
-   通过 libP2P 网络传播 Beacon 区块和 Attestation
    
-   支持同步委员会（Sync Committees）以服务轻客户端
    

主流共识客户端：Lighthouse（Rust）、Prysm（Go）、Teku（Java）、Nimbus（Nim）、Lodestar（TypeScript）

  
  

**2.4 Engine API：两层协同**

**执行层与共识层通过本地 Engine API（JSON-RPC）通信：**

-   共识层发起：engine\_forkchoiceUpdated，通知 EL 当前最优链头
    
-   共识层发起：engine\_newPayload，传递新执行载荷（交易列表）
    
-   执行层返回：执行结果与状态根，由 CL 纳入 Beacon 区块
    

  
  

| ⚠️重要：客户端多样性客户端多样性是网络韧性的关键。若任何单一客户端占比超过 66%，一旦出现严重 Bug，将威胁整个网络的最终确定性。多客户端架构是以太坊的核心安全设计。 |

  
  

**三、设计理念 (Design Rationale)**

**以太坊的每个设计决策背后都有深刻的权衡与哲学，下面梳理核心原则与关键技术选择的理由。**

**3.1 五大核心设计原则**  

| 简洁性 Simplicity | 协议尽量简单，即使以牺牲部分存储或时间效率为代价。普通程序员应能理解和实现规范。复杂度越低，审计越容易，安全漏洞越少，治理的合法性也越强。 |
| 通用性 Universality | 以太坊提供图灵完备的 EVM，不内置特定应用逻辑。任何可编程的事情都可在以太坊上实现，协议本身不偏袒任何特定用例。 |
| 模块化 Modularity | 协议各部分尽量解耦，可独立升级。Patricia Trie、RLP、执行层/共识层均设计为可独立使用的组件。创新优先推至 L2 或客户端层，而非污染 L1 规范。 |
| 敏捷性 Agility | 协议应愿意根据新发现进行修改，以太坊通过硬分叉机制持续演进。低层协议保持长期稳定，高层协议则灵活迭代。 |
| 无歧视性 Non-discrimination | 协议不主动限制或阻止特定类别的使用，监管机制应直接针对具体危害，而非反对特定应用场景。以太坊是中立基础设施。 |

  
  

**3.2 账户模型 vs UTXO 模型**

**以太坊选择账户模型而非比特币的 UTXO 模型，主要理由：**

-   简洁性：账户模型对程序员更直观，UTXO 方案在支持复杂 DApp 时极为繁琐
    
-   轻客户端友好：轻节点可通过状态树路径直接访问账户数据，UTXO 引用会随每笔交易变化
    
-   状态存储效率：账户模型下同一账户的多次交易不产生冗余 UTXO，存储更高效
    

  
  

**3.3 EVM 设计理念**

**EVM 的设计目标：**

-   简洁：尽量少的操作码和数据类型，无类型系统（以有符号/无符号 opcode 替代）
    
-   完全确定性：规范不留任何歧义，Gas 计量精确到每一步计算
    
-   紧凑性：字节码尽量小（避免 C 程序默认 4000 字节基础开销）
    
-   安全的 Gas 模型：操作码成本模型使 VM 不可被经济攻击
    
-   专用优化：原生支持 20 字节地址、32 字节大数运算、自定义密码学
    

  
  

**3.4 Merkle Patricia Trie（MPT）**

**以太坊的世界状态存储在 MPT 中，设计权衡：**

-   使用 sha3(key) 作为安全树键：防止攻击者构造深度 64 层的不利链条进行 DDoS
    
-   区分终止与非终止节点：增强通用性，使 MPT 实现可被其他协议复用
    
-   不区分空值与不存在：以简洁性换取少量通用性
    

  
  

**3.5 长期稳定性原则**

| 核心思路低层协议应构建得十年内无需修改，所有必要创新都在更高层（客户端实现或 L2 协议）发生。当复杂度不可避免时，优先顺序为：L2 协议 > 客户端实现 > 协议规范。 |

  
  

**四、以太坊发展历史 (History)**

**以太坊从 2013 年一份白皮书，成长为全球最重要的智能合约平台，经历了多个关键里程碑。**

  
  

**4.1 创世与早期（2013–2015）**

-   2013 年底：Vitalik Buterin 发布以太坊白皮书，提出"可编程区块链"概念
    
-   2014 年 1 月：在迈阿密比特币大会上正式宣布，Gavin Wood 加入并编写黄皮书（EVM 技术规范）
    
-   2014 年 7–8 月：42 天公开预售，募得约 31,591 BTC（约 1,800 万美元），分发 6,000 万 ETH
    
-   2014 年 6 月：以太坊基金会在瑞士楚格注册成立
    
-   2015 年 7 月：Frontier 上线，以太坊主网正式启动（仅面向技术用户，Gas 上限 5,000）
    

  
  

**4.2 主要升级时间线**

  
  

| 时间 | 升级名称 | 核心内容 |
| 2015.07 | Frontier | 主网上线，面向开发者，PoW 挖矿，Gas 上限 5,000 |
| 2016.03 | Homestead | 第一个计划性硬分叉；移除中心化Canary 合约；引入 Mist 钱包；Solidity 新特性 |
| 2016.07 | DAO Fork | 应对 DAO 黑客事件（~5,000 万美元 ETH 被盗）；争议性硬分叉回滚交易；催生 Ethereum Classic |
| 2017.10 | Byzantium | Metropolis 第一阶段；区块奖励从 5 ETH → 3 ETH；支持 zk-SNARKs 密码学原语；引入 Difficulty Bomb |
| 2019.02 | Constantinople | Metropolis 第二阶段；区块奖励从 3 ETH → 2 ETH；多项 EVM 效率优化 |
| 2019.12 | Istanbul | 优化 EVM Gas 成本；为 Layer 2（zk-rollup）奠定基础 |
| 2020.12 | Beacon Chain | PoS 共识层上线（独立运行），验证者开始质押 32 ETH，但尚未与主网合并 |
| 2021.08 | London / EIP-1559 | 革命性 Gas 费用改革：引入基础费用销毁机制（ETH 通缩）；改善交易费可预期性 |
| 2022.09 | The Merge（巴黎） | 执行层与共识层合并；PoW → PoS 转换；能耗降低约 99.95%；以太坊历史上最重大升级 |
| 2023.04 | Shapella | Shanghai（EL）+ Capella（CL）；首次允许验证者提取质押 ETH；完成 PoS 完整闭环 |
| 2024.03 | Dencun | 引入 EIP-4844「Proto-Danksharding」；Blob 交易大幅降低 L2 Gas 费（降幅 80-90%） |
| 2025.05 | Pectra | Prague + Electra；EIP-7702 账户抽象（EOA 获得智能合约能力）；验证者上限升至 2,048 ETH；扩展Blob 吞吐量 |

  
  

**4.3 重大历史事件**

**DAO 事件（2016）**

**DAO（去中心化自治组织）于 2016 年成为以太坊最大众筹项目，28 天内募集约 1.5 亿美元。6 月，攻击者利用重入漏洞提取约 5,000 万美元 ETH。社区就是否回滚交易产生严重分歧，最终投票通过硬分叉，分裂为ETH（回滚）和 ETC（不回滚）两条链。这一事件引发了关于区块链不可变性与社区治理的深刻讨论。**

  
  

**The Merge（2022）**

**The Merge 是以太坊史上最复杂、最受瞩目的升级。在不中断主网运行的情况下，将 PoW 执行层与 PoS 共识层（Beacon Chain，2020 年已独立运行）无缝合并。这一【在飞行中换引擎】的壮举几乎消除了以太坊的能源消耗，同时为进一步的分片和扩容升级奠定了基础。**

  
  

**4.4 以太坊路线图展望**

**Vitalik 提出的长期路线图以形象命名，围绕"可扩展、安全、去中心化"三角展开：**

  
  

-   The Merge ✅：完成 PoS 过渡
    
-   The Surge：数据分片（Danksharding），将 L2 TPS 提升到万级
    
-   The Scourge：抗 MEV 机制，保护用户和去中心化
    
-   The Verge：Verkle Trees 替换 MPT，使无状态客户端成为可能
    
-   The Purge：历史数据清理（EIP-4444），降低节点运营门槛
    
-   The Splurge：其他零散改进，包括账户抽象、隐私增强等  
    

| 2025+ 展望Vitalik 在 2025 年提出「简化 L1」愿景：未来 5 年内，将以太坊共识关键代码的复杂度降至接近比特币的水平，以提升安全性和去中心化。Beam Chain（新共识层设计）和 RISC-V EVM 替换方案是这一方向的重要探索。 |

  

本笔记基于 [_epf.wiki_](http://epf.wiki) _Protocol Wiki_ 内容整理，涵盖 _prehistory / architecture / design-rationale / history_ 四个模块
<!-- DAILY_CHECKIN_2026-04-08_END -->
<!-- Content_END -->
