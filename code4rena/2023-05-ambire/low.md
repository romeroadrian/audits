# Report

- Low Issues (2)
- Non Critical Issues (4)
## Low Issues

| |Issue|
|-|:-|
| [L-1](#L-1) | AmbireAccountFactory can deploy backdoored code  |
| [L-2](#L-2) | `splitSignature()` does not validate length of signature |


### <a name="L-1"></a>[L-1] AmbireAccountFactory can deploy backdoored code

The AmbireAccountFactory deploys new wallets by creating a contract using an arbitrary code given as a parameter. This can be used in a MITM attack to backdoor the wallet and install another privilege that would give the attacker control over the wallet.

Although this action may go unnoticed it is important to note that backdooring the code would change the address and could be early detected, hence the low severity.

### <a name="L-2"></a>[L-2] `splitSignature()` does not validate length of signature

https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/SignatureValidator.sol#L29-L36

The `splitSignature()` does not validate that the length of the signature is greater than 0 and may end up causing an underflow in the unchecked block.

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 3 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 2 |
| [NC-3](#NC-3) | Use `uint256` instead of the `uint` alias | 1 |
| [NC-4](#NC-4) | Unneeded explicit return | 1 |

### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (3)*:
```solidity
File: src/AmbireAccount.sol

4: import './libs/SignatureValidator.sol';

```

```solidity
File: src/AmbireAccountFactory.sol

4: import './AmbireAccount.sol';

```

```solidity
File: src/libs/SignatureValidator.sol

4: import './Bytes.sol';

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (2)*:
```solidity
File: src/AmbireAccount.sol

13: 	mapping(address => bytes32) public privileges;

15: 	mapping(bytes32 => uint) public scheduledRecoveries;

```

### <a name="NC-3"></a>[NC-3] Use `uint256` instead of the `uint` alias
Prefer using the `uint256` type definition over its `uint` alias.

*Instances (1)*:
```solidity
File: src/AmbireAccount.sol

15: 	mapping(bytes32 => uint) public scheduledRecoveries;

```

### <a name="NC-4"></a>[NC-4] Unneeded explicit return

The explicit return can be omitted as the function is using named return data.

- https://github.com/AmbireTech/ambire-common/blob/5c54f8005e90ad481df8e34e85718f3d2bfa2ace/contracts/libs/Bytes.sol#L32
