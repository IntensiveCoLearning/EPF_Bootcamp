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
