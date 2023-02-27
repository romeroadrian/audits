## Functions visibility in `Pool` contract

Both `withdraw` and `transferFrom` visibility can be changed to external since neither function is called internally.

## Change visibility for `isInternal` and `remainingETH` variables in Exchange contract to private

Both of these functions are used internally in the contract to prevent a direct external call to `_execute` and to track how much ETH is consumed. There shouldn't be any reason to expose them with getters. 

## Initial values in storage variables of Exchange contract

The Exchange contract initializes the `isInternal` and `remainingETH` variables in the field declaration, however these values vwon't have any effect since the contract is an upgradable contract. Reporting this as low since both initial values are the default values (`false` and `0`) and the effect will be the same as if they were correctly initialized.

## Creation dependency loop between Exchange and Pool contracts

The Exchange contract supports payments using funds previously deposited into the Pool contract. It references the Pool using a constant:

```
address public constant POOL = 0xF66CfDf074D2FFD6A4037be3A669Ed04380Aef2B;
```

In a similar way, the Pool contract needs to know the Exchange contract to control access to funds transfers. It defines the Exchange contract using a constant too:

```
address private constant EXCHANGE = 0x707531c9999AaeF9232C8FEfBA31FBa4cB78d84a;
```

This will create a dependency loop where contract deployments depend on each other. 


## `bulkExecute` function is missing in the `IExchange` interface

The interface for the Exchnage contains the `execute` function but lacks the definition for the `bulkExecute` function.
