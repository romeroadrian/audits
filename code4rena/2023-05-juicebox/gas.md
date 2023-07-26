# JBXBuybackDelegate.sol contract

- Use Solidity 0.8.20 with Shanghai EVM target to take advantage of new `PUHS0` opcode when deploying to mainnet.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2

- Storage variable `mintedAmount` doesn't need to be initialized to 1, as this value will be eventually set in `payParams()`.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L106
  
- Storage variable `reservedRate` doesn't need to be initialized to 1, as this value will be eventually set in `payParams()`.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L113
  
- There is really no need and no benefit of calculating the minimum token amount in the `payParams()` function as `_quote` and `_slippage` are both fixed params that are received as params. Simply calculate `_quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)` off-chain and send that as the `minOutputAmount` for the swap operation.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L197

- There is no need to reset the `mintedAmount` storage variable to 1 as this value will be overwritten the next time `payParams()` is called.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L189

- There is no need to reset the `reservedRate` storage variable to 1 as this value will be overwritten the next time `payParams()` is called.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L193

- The `_amount` argument in the call to `addToBalanceOf()` can be set to zero to save gas in calldata, as the `addToBalanceOf()` implementation will read and overwrite the amount using the value from callvalue.
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L348-L350  

- Instead of calculating the `_nonReservedToken` by doing `JBConstants.MAX_RESERVED_RATE - _reservedRate`, simplify the calculations by first calculating `_reservedToken` using `_reservedRate` (i.e. `PRBMath.mulDiv(_amountReceived, _reservedRate, JBConstants.MAX_RESERVED_RATE)`) and then calculating `_nonReservedToken` as the difference of `_amountReceived` and `_reservedToken`.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L278-L283

- Variable `_nonReservedTokenInContract` is the same as `_nonReservedToken` since it is the difference between `_amountReceived` and `_reservedToken`. There is no need to calculate this again.  
  https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312
