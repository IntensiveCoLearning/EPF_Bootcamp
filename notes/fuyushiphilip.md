---
timezone: UTC+8
---

# FYS

**GitHub ID:** fuyushiphilip

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->
**Fork Choice · Finality · Slot · Epoch · Checkpoint · Slashing**

-   **Fork Choice**: Determines which chain is the correct one when temporary disagreements occur.
    
-   **Finality**: Provides an irreversible guarantee that a block will never be reverted or reorganized.
    
-   **Slot**: A 12-second time unit in which a single validator is assigned to propose a block.
    
-   **Epoch**: A group of 32 slots (6.4 minutes) used as the fundamental period for validator rewards and finality checks.
    
-   **Checkpoint**: The block at the start of each epoch, which serves as the target for finality votes.
    
-   **Slashing**: Penalizes malicious or dishonest validators by burning a portion of their staked ETH.  
      
    Time is divided into **slots** (12 seconds) and **epochs** (32 slots). The boundary blocks of each epoch are **checkpoints**. Validators vote on checkpoints, and when two consecutive checkpoints receive over 2/3 agreement, **finality** is achieved. If the chain temporarily diverges, the **fork choice** rule (LMD-GHOST) is used to select the correct chain. Malicious votes are penalized through **slashing**.  
      
    

![deepseek_mermaid_20260409_50c4c4.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-09-1775742008351-deepseek_mermaid_20260409_50c4c4.png)
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->

**EIP-1559**  
  
Before EIP-1559, Ethereum used a First-Price Auction mechanism. Users would set their own gasPrice, and miners (later validators) would prioritize including transactions with the highest bids. This led to several serious issues:

-   Extreme fee volatility, with users easily overbidding or guessing wrong during peak times, resulting in transaction failures.
    
-   Poor user experience and difficulty estimating gas.
    
-   Miners had an incentive to manipulate block size to drive up fees (although this didn't happen often in practice, it was theoretically possible).
    

EIP-1559 introduced Type 2 transactions (EIP-1559 transactions), splitting fees into three components:

-   **Base Fee**: Automatically calculated by the protocol, adjusted per block based on the gas usage of the previous block. The target block size is approximately 15 million gas (with a maximum limit of 30 million gas). If the previous block exceeds the target, the base fee increases by up to 12.5%; if it falls below the target, the base fee decreases by up to 12.5%. This portion of the fee is completely burned, permanently removed from the system, and given to no one.
    
-   **Priority Fee (Tip)**: Set by the user (maxPriorityFeePerGas). This portion goes directly to the validator as an incentive for including the transaction.
    
-   **Max Fee Per Gas**: The total upper limit set by the user. The actual payment = Base Fee + Priority Fee (any surplus is refunded).  
      
      
    new\_base\_fee = old\_base\_fee × (1 + 0.125 × (gas\_used - target\_gas) / target\_gas)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->


![Screenshot 2026-04-07 at 7.07.19 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560105106-Screenshot_2026-04-07_at_7.07.19_PM.png)![Screenshot 2026-04-07 at 7.15.48 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560560940-Screenshot_2026-04-07_at_7.15.48_PM.png)![Screenshot 2026-04-07 at 7.15.11 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560527376-Screenshot_2026-04-07_at_7.15.11_PM.png)
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
