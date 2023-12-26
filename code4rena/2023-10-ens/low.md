# Report

## Summary

### Low Issues

Total of **5 issues**:

|ID|Issue|
|:--:|:---|
| [L-1](#l-1-multi-delegate-interface-is-unnecessarily-confusing) | Multi delegate interface is unnecessarily confusing |
| [L-2](#l-2-design-doesnt-allow-delegations-and-reimbursements-at-the-same-time) | Design doesn't allow delegations and reimbursements at the same time |
| [L-3](#l-3-missing-events-for-delegation-and-reimbursement) | Missing events for delegation and reimbursement |
| [L-4](#l-4-proxy-approval-can-eventually-be-exhausted) | Proxy approval can eventually be exhausted |
| [L-5](#l-5-proxy-deployment-should-check-the-deployed-account-has-code) | Proxy deployment should check the deployed account has code |

### Non Critical Issues

Total of **1 issues**:

|ID|Issue|
|:--:|:---|
| [NC-1](#nc-1-use-require-instead-of-assert-statements) | Use `require` instead of `assert` statements |

## Low Issues

### <a name="L-1"></a>[L-1] Multi delegate interface is unnecessarily confusing

The `sources` and `targets` arrays are used to represent the three main operations supported by the protocol, new delegations, transfers of delegated tokens, and reimbursements. 

This design is implemented by considering transfers to elements whose indices are present in both arrays, new delegations as indices in the `sources` array not present in the `targets` array, and reimbursements as indices in the `targets` array not present in the `sources` array.

This interface is unnecessarily confusing and error prone. Consider switching to a simpler interface to list the intended actions more clearly. For example, this can be an array of operations, where each operation is an struct that has an enum to indicate the action (new delegation, transfer, reimbursement) along with the proper fields that serve as the arguments of the operation.

### <a name="L-2"></a>[L-2] Design doesn't allow delegations and reimbursements at the same time

Delegations are represented by specifying a target address, while reimbursements are represented by specifying a source address. But both operations cannot be combined in the same call, since if sources and targets are provided at the same time, these are considered transfers of delegations.

The current design doesn't allow to specify new delegations and reimbursements in the same call to `delegateMulti()`.

### <a name="L-3"></a>[L-3] Missing events for delegation and reimbursement

Transfer of delegations between a source and target fire the `DelegationProcessed` event, but the other actions don't emit any log.

Consider also adding events for new delegations or reimbursements.

### <a name="L-4"></a>[L-4] Proxy approval can eventually be exhausted

When deployed, proxies grant its caller (the ERC20MultiDelegate contract) an approval for the max value of the `uint256` type.

This allowance can be eventually depleted if multiple operations are executed in the same proxy (delegatee), without the possibility of renewing this approval, leading to a potential denial of service. This would require lots of transfers for large amounts, hence the low severity.

### <a name="L-5"></a>[L-5] Proxy deployment should check the deployed account has code

https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L185

The conditions to deploy a proxy to represent a delegatee is based on the codesize of the expected address account.

```solidity
173:     function deployProxyDelegatorIfNeeded(
174:         address delegate
175:     ) internal returns (address) {
176:         address proxyAddress = retrieveProxyContractAddress(token, delegate);
177: 
178:         // check if the proxy contract has already been deployed
179:         uint bytecodeSize;
180:         assembly {
181:             bytecodeSize := extcodesize(proxyAddress)
182:         }
183: 
184:         // if the proxy contract has not been deployed, deploy it
185:         if (bytecodeSize == 0) {
186:             new ERC20ProxyDelegator{salt: 0}(token, delegate);
187:             emit ProxyDeployed(delegate, proxyAddress);
188:         }
189:         return proxyAddress;
190:     }
```

Lines 179-182 fetch the `extcodesize` of the proxy address, and line 185 checks if the size is zero in order to prevent redeploying the proxy.

It is important to note that, technically, contracts can be deployed without storing any code. If the contract's creation code returns empty, the account will be created with empty code.

This means that the next time `deployProxyDelegatorIfNeeded()` is called, the `bytecodeSize` variable will be zero, leading to a redeploying of the proxy, which will fail since the account has been already created, causing a denial of service.

Consider adding a check to ensure proxies are created with actual code, so successive calls to `deployProxyDelegatorIfNeeded()` are ensured to be checked correctly.

```diff
        if (bytecodeSize == 0) {
            new ERC20ProxyDelegator{salt: 0}(token, delegate);
            
+           assembly {
+             bytecodeSize := extcodesize(proxyAddress)
+           }
+           require(bytecodeSize > 0);
            
            emit ProxyDeployed(delegate, proxyAddress);
        }
```

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Use `require` instead of `assert` statements

- https://github.com/code-423n4/2023-10-ens/blob/main/contracts/ERC20MultiDelegate.sol#L131

Checks or preconditions in functionality should use `require` instead of `assert`.

