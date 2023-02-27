## Reorder guard in `permit` function of `GraphTokenUpgradeable`

The require in line 95 can be safely moved to the top of the function to early exit the function and save some gas on the user.

```
require(_deadline == 0 || block.timestamp <= _deadline, "GRT: expired permit");
```

## The funcion `bridgeBurn` could directly call `_burn` in `L2GraphToken`

The function `bridgeBurn` present in the `L2GraphToken` contract ends up calling the public function `burnFrom` of `ERC20Burnable`, which checks the allowance requires an explicit approval from the account to be able to burn those tokens. 

The `bridgeBurn` function could instead call `_burn` directly, since this happens in the same token contract, and should be safe since this call is part of the L2 -> L1 bridging process and is restricted to be called only from the gateway. This would reduce the function logic and save the user an explicit approval before bridging the tokens.
