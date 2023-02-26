## Double call to `_getReceiver` in `royaltyInfo` function of `PA1D` contract

The function `_getReceiver` which loads the receiver from storage is called twice in the body of `royaltyInfo`. Consider storing the first call in a local variable to prevent a second call.

## Double call to `_getReceiver` in `getRoyalties` function of `PA1D` contract

The function `_getReceiver` which loads the receiver from storage is called twice in the body of `getRoyalties`. Consider storing the first call in a local variable to prevent a second call.

## Double check of token existence in `bridgeIn` function of `HolographERC721` contract

The code checks that the token doesn't exist in line 404 and then calls the `_mint` which executes the exact same check in line 817.

## Unnecessary swap when removing last element from array in `_removeTokenFromAllTokensEnumeration` of `HolographERC721` contract

The code removes a token from the `_allTokens` array by swapping it for the last element and calling pop. If the element to be removed is that last element then the swap is not needed.

## The `_transferFrom` function in the `HolographERC721` contract removes and adds the token again to the enumeration

The `_transferFrom` function will call `_removeTokenFromOwnerEnumeration` which will remove the token from the `_allTokens` array, just to add it back again in the `_addTokenToOwnerEnumeration` function. Consider refactoring the code to save gas in this scenario, where the token just needs to be removed from the sender and added to the recipient of the transfer, there's no need to modify the `_allTokens` array during a transfer.

## Unchecked math gas savings in `HolographERC20` contract

- In the `_burn` function, the update to the `_totalSupply` variable can be placed inside the unchecked block, since `accountBalance <= _totalSupply` and `amount <= accountBalance` (due to the check above). https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L633
- In the `_mint` function, the update to the `_balances[to]` variable can be placed inside an unchecked block, since `balance <= totalSupply` and the overflow check should be covered in line 685. https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L686
- In the `_transfer` function, the update to the `_balances[recipient]` variable can be placed inside the unchecked block, since the sum of all balances are limited by the `totalSupply` and an overflow here should be impossible. https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L702

## Move the `_blockTime` variable to a constant in the `HolographOperator` contract

This storage variable is initialized with a literal value of 60 and doesn't have a setter to override its value. Consider moving it to a constant to avoid loading it from storage and save gas.

## Store value of `_operatorTempStorageCounter` variable locally to avoid a re-read in function `crossChainMessage` of the `HolographOperator` contract

The `_operatorTempStorageCounter` variable is incremented in line 495, and then re-read from storage a second time in line 517 and a third time in line 524. Save the value in a local variable after the increment and use that value later

```
uint256 storageCounter = ++_operatorTempStorageCounter;
```

## `bridgeIn` function in `HolographFactory` does an external call to a function in the same contract

The `bridgeIn` function in the `HolographFactory` contract calls the `deployHolographableContract` externally by doing `HolographFactoryInterface(address(this)).deployHolographableContract(...)`. Consider changing `deployHolographableContract` to be public so it can be called internally to save gas.
