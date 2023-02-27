## Split `require` that use `&&` into multiple statements

https://github.com/pooltogether/ERC5164/blob/5647bd84f2a6d1a37f41394874d567e45a97bf48/src/ethereum-optimism/EthereumToOptimismExecutor.sol#L80-L84

This require checks that caller is the bridge and that the original messages comes from the relayer on the other side of the chain. Consider splitting the require into two separate statements to save gas.

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas. 

## Consider using immutable variables for relayer and executor in Arbitrum and Optimism contracts

In both chains, the relayer needs a reference to the executor and the executor needs a reference the relayer. This is implemented using a setter (which is intented to be used only once) and a storage variable to hold the address.

Since in both cases the address is only set once and can't be modified, consider using an immutable variable which will save an sload in each side of the bridge for each bridged message.

The dependency loop at construction time (relayer needs executor and executor needs relayer) can be solved by using precomputed addresses and deploying the contracts using a factory along with create2. 
