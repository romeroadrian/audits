## Use short-circuit boolean evaluation in if guard

Expressions that are part of an if guard are evaluated before the statement, causing unnecessary gas costs in cases where not all conditions need to be evaluated to select the branch. 

Occurrences: 2

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/BaseAbstract.sol#L44

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/BaseAbstract.sol#L55

## Store locally value of `rewardsPool.getRewardsCycleCount()` in `ClaimNodeOp` contract

The function `rewardsPool.getRewardsCycleCount()` is called 3 times in lines 61, 65 and 68. Store locally the result of the first call and reuse this later.

https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ClaimNodeOp.sol#L61

## Usage of safe unchecked math

- In `ClaimNodeOp.claimAndRestake`, line 102 can be wrapped in an unchecked block due to the if guard in line 95.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/ClaimNodeOp.sol#L102

- In `Vault.withdrawAVAX`, line 75 can be unchecked due to the if guard in line 71.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L75

- In `Vault.transferAVAX`, line 99 can be unchecked due to the if guard in line 95.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L99

- In `Vault.withdrawToken`, line 155 can be unchecked due to the if guard in line 151.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L155

- In `Vault.transferToken`, line 187 can be unchecked due to the if guard in line 183.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/Vault.sol#L187

- In `TokenggAVAX.depositFromStaking`, line 149 can be unchecked due to the if guard in line 144.  
https://github.com/code-423n4/2022-12-gogopool/blob/main/contracts/contract/tokens/TokenggAVAX.sol#L149
