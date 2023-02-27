## Disable initializers in implementation contracts of `PirexRewards`

Even though the contract doesn't expose any `delegatecall` or way to call `selfdestruct` (it uses transparent proxies instead of UUPS), consider adding a constructor to disable initializers in the implementation contract of `PirexRewards`.

```solidity
constructor() {
    _disableInitializers();
}
```

## Call to `super` instead of duplicating code in `transfer` and `transferFrom` functions of `PxERC20` contract

Both functions have the same code as their respective base ERC20 implementation with an added code to accrue rewards. Call the base implementation instead of duplicating all the code. For example:

```solidity
function transfer(address to, uint256 amount)
    public
    override
    returns (bool)
{
    bool result = super.transfer(to, amount);
    
    // Accrue rewards for sender, up to their current balance and kick off accrual for receiver
    pirexRewards.userAccrue(this, msg.sender);
    pirexRewards.userAccrue(this, to);    

    return result;
}
```

## Incorrect comment in `AutoPxGlp` constructor

The comment in line 86 reads:

> // Approve the Uniswap V3 router to manage our base reward (inbound swap token)

However this seems to be a badly copy pasted comment from the AutoPxGmx contract. The approval is to the PirexGmx contract in order to compound the rewards.

## Call to `super` instead of duplicating code in `withdraw` and `redeem` functions of `AutoPxGmx` contract

Both functions have the same code as their respective base ERC4626 implementation with a prepended code to compound the vault. Call the base implementation instead of duplicating all the code. For example:

```solidity
function withdraw(
    uint256 assets,
    address receiver,
    address owner
) public override returns (uint256 shares) {
    // Compound rewards and ensure they are properly accounted for prior to withdrawal calculation
    compound(poolFee, 1, 0, true);
    
    shares = super.withdraw(assets, receiver, owner);
}
```
