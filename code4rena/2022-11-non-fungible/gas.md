## Unchecked math in `withdraw` function of `Pool` contract

Line 46 can be done using unchecked math due to the require guard in line 45 (`_balances[msg.sender] >= amount`).

https://github.com/code-423n4/2022-11-non-fungible/blob/main/contracts/Pool.sol#L46

## Unchecked math in `_transfer` function of `Pool` contract

Line 73 can be done using unchecked math due to the require guard in line 71 (`_balances[from] >= amount`).

Line 74 can be added to the unchecked math block since it will require an unexisting amount of ETH to possibly overflow it.

https://github.com/code-423n4/2022-11-non-fungible/blob/main/contracts/Pool.sol#L73
