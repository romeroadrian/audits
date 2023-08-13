# MathUtils library

- Function `calculateCurrentInterest()` uses unnecessary intermediary WAD scaling resulting in unneeded multiplications and divisions as a result of `wadMul` and `wadDiv`.  
  The calculation can be simplified as `(principal * rateBps * (block.timestamp - loanStartTime)) / (_BASIS_POINTS * 365 days)`.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/libraries/math/MathUtils.sol#L19
 
- Function `calculateCurrentAuctionPrice()` uses unnecessary intermediary WAD scaling resulting in unneeded multiplications and divisions as a result of `wadMul` and `wadDiv`.  
  The calculation can be simplified as `(price * auctionElapsed / auctionDuration`.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/libraries/math/MathUtils.sol#L36

- Function `calculateTreasuryProportion()` uses unnecessary intermediary WAD scaling resulting in unneeded multiplications and divisions as a result of `wadMul` and `wadDiv`.  
  The calculation can be simplified as `(interest * rateBips / _BASIS_POINTS`.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/libraries/math/MathUtils.sol#L50

# ParticleExchange contract

- Storage variables `_treasuryRate` and `_treasury` could benefit from being packed together in a single slot as these are used in conjunction. This could be achieved by changing `_treasuryRate` to be a `uint24` (which can safely fit the maximum rate of 100_000) and reducing the size of `_treasury` to uint232 (which can still represent a huge amount of ETH).  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L22-L23

- In `withdrawEthWithInterest()` there are potentially multiple calls to the lender to send ETH. Group these together in order to execute a single `transfer()` call.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L212  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L222

- In the `_withdrawAccountInterest()` function the `_treasuryRate` variable is read twice from storage. Consider caching it locally to execute a single SLOAD.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L237  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L238
  
- In the `_withdrawAccountInterest()` function, the math in `_treasury += treasuryInterest` can be unchecked as it would be practically impossible to overflow the size of the `_treasury` variable.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L239

- In the `_withdrawAccountInterest()` function, the math in `interest -= treasuryInterest` can be unchecked as `treasuryInterest` is a portion of the `interest` (i.e. `treasuryInterest <= interest`).  
https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L240

- If marketplaces are trusted entities then WETH approval can be done once for an infinite amount instead of approving a specific amount in each call to `_execBuyNftFromMarket()`.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L418

- ERC721 transfers can be safely executed by calling `transferFrom()` instead of `safeTransferFrom()` when the recipient is the exchange contract in order to avoid the unneeded callback and save gas.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L57  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L499  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L731

- In the `stopLoanAuction()` function the check `lien.loanStartTime == 0` is unneeded this is already implied by the check below of `lien.auctionStartTime == 0` (i.e. if `lien.auctionStartTime != 0` is true then it already implies that the lien is taken).  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L659
  
- In the `auctionBuyNft()` function, the math in `lien.credit + lien.price - payableInterest - amount` can be unchecked as `amount` is lower than or equal to `currentAuctionPrice`, and `currentAuctionPrice` is a portion of `maxSpendable` (`lien.credit + lien.price - payableInterest`), which means that `lien.credit + lien.price - payableInterest` is safe (as it didn't overflow in line 699) and `lien.credit + lien.price - payableInterest - amount` is also safe as `amount <= lien.credit + lien.price - payableInterest`.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L742

- In the `withdrawTreasury()` function, the `_treasury` storage variable is read twice from storage. Consider caching it locally to execute a single SLOAD.  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L809  
  https://github.com/code-423n4/2023-05-particle/blob/main/contracts/protocol/ParticleExchange.sol#L813
