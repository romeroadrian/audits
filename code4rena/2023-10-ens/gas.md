# ERC20ProxyDelegator contract

- The contract's constructor can be safely marked as `payable` to save gas costs while deploying the proxy. This contract is always created through ERC20MultiDelegate without any callvalue.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L16  
  
- Both `token.approve()` and `token.delegate()` can be executed as low level calls to avoid checks introduced by the compiler. This should be safe since the token is a known token implementation coming from the ERC20MultiDelegate contract.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L17-L18
  
# ERC20MultiDelegate contract

- The `token` variable can be `immutable` instead of storing it in contract storage. This value is assigned in the constructor and isn't mutable in the implementation.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L28  

- The computation of the maximum between the arrays length (i.e. `Math.max(sourcesLength, targetsLength)`) is repeated in `_delegateMulti()`. Consider executing this once and saving the result in a local variable.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L80  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L87  
  
- Casting of `source` and `target` addresses can be moved inside each branch of the `if` to avoid unnecessary operations. Only in the first branch of the `if` both castings are required, in the other two cases only one casting is needed.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L90-L95  

- The calculation of the minimum value between the arrays length (i.e. `Math.min(sourcesLength, targetsLength)`) is constant in the for-loop iteration and can be extracted outside to execute it only once and save gas costs.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L98  
  
- In `transferBetweenDelegators()`, the address for the proxy belonging to the target (`retrieveProxyContractAddress(token, to)`) has been already calculated previously in `deployProxyDelegatorIfNeeded(target)` (line 133). Consider using this result instead of recalculating it again.  
  https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L169  
