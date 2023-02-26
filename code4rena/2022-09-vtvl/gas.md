## Simplify `finalVestedAmount` in `revokeClaim`

`finalVestedAmount` can be simplified to the sum of linear + cliff amounts:

```
uint112 finalVestAmt = _claim.linearVestAmount + _claim.cliffAmount;
```

Since `revokeClaim` operates on active claims, the final vested amount should be equivalent to adding both amounts. 

Since this is only usage of `finalVestedAmount` the function can be removed.

https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/VTVLVesting.sol#L422
