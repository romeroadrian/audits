## Use `_disableInitializers()` in `Escher721` contract

The `Escher721` contract is used by cloning the implementation. Consider adding the `_disableInitializers()` call in the constructor as precautionary measure.

## Remove `_burn` override in `Escher721` contract

The contract overrides the `_burn` function only to call super. Consider removing this unneeded override. Project will compile fine with Solidity 0.8.17 and all tests pass if this override is removed.

## Rename `Start` event used in sales contracts

Consider renaming the `Start` event for the Sales contract since technically sales have a "start time" that can be in the future and the event is emitted at creation time.

## Validate sale parameters in `FixedPriceFactory` contract

In `createFixedSale` function:

- Validate `sale.price > 0`
- Validate `sale.saleReceiver != address(0)`

## Invalid initialization of storage variable `amountSold` in `LPDA` contract

The `amountSold` variable is initialized during the variable definition `uint48 public amountSold = 0;`. This is equivalent to using the constructor, which in this case is invalid since this contract is created using clones.

Assigning this a low severity since the assigned value is 0 which matches the default value and the issue shouldn't have a negative effect. However, consider removing this initialization.

## Add a getter for sale `endTime` in `LPDA` contract

Add a getter to expose the `endTime` value of the sale.

```solidity
function endTime() public view returns (uint256) {
    return sale.endTime;
}
```

## `Buy` event is emitted with old data in `OpenEdition.buy` function

The `Buy` event is emitted using the `temp` variable which has the old `sale.currentId` (value is updated in line 69 directly to storage).

## Validate sale parameters in `OpenEditionFactory` contract

In `createOpenEdition` function:

- Validate `sale.price > 0`
- Validate `sale.saleReceiver != address(0)`

## Fee beneficiary can DoS and block the sale end process

The sale contracts uses the `transfer` function to distribute fees to the fee receiver defined in its factory contract. If the fee receiver is a contract which executes some logic on `receive` or `fallback`, it is possible that the call fails due to gas costs (transfer calls are bounded to 2300 units of gas) or directly by a revert on the call. If the `transfer` call fails, then the whole procedure will revert.

Reporting this as a low severity because fee receiver address will likely be in control of the protocol owner and can eventually be updated in the factory contracts.
