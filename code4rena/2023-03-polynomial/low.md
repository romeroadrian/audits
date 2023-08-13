# Report

- Non Critical Issues (2)
- Low Issues (8)

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Bool expression compared to literal value | 1 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 8 |

### <a name="NC-1"></a>[NC-1] Bool expression compared to literal value
Bool expressions do not need to be compared against a literal value. For example, `aBoolExpression == true` can be directly stated as `aBoolExpression`, or `aBoolExpression == false` as `!aBoolExpression`.

*Instances (1)*:
```solidity
File: src/utils/PauseModifier.sol

12:         require(systemManager.isPaused(key) == false);

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (8)*:
```solidity
File: src/KangarooVault.sol

151:     mapping(uint256 => QueuedDeposit) public depositQueue;

154:     mapping(uint256 => QueuedWithdraw) public withdrawalQueue;

```

```solidity
File: src/LiquidityPool.sol

145:     mapping(uint256 => QueuedDeposit) public depositQueue;

148:     mapping(uint256 => QueuedWithdraw) public withdrawalQueue;

```

```solidity
File: src/ShortCollateral.sol

34:     mapping(bytes32 => Collateral) public collaterals;

37:     mapping(uint256 => UserCollateral) public userCollaterals;

```

```solidity
File: src/ShortToken.sol

26:     mapping(uint256 => ShortPosition) public shortPositions;

```

```solidity
File: src/SystemManager.sol

50:     mapping(bytes32 => bool) public isPaused;

```


## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 10 |
| [L-2](#L-2) | KangarooVault should initialize `systemManager`  | - |
| [L-3](#L-3) | LiquidityPool `withdraw` should use `safeTransfer` | - |
| [L-4](#L-4) | LiquidityPool `processWithdraws` overwrites `returnedAmount` for withdrawals processed in multiple steps | - |
| [L-5](#L-5) | Missing event for important parameter change	| 1 |
| [L-6](#L-6) | SystemManager initialization can be front-runned	| - |
| [L-7](#L-7) | Synthetix market can be empty in `refreshSynthetixAddresses`	| - |
| [L-8](#L-8) | VaultToken `setVault` initialization can be front-runned	| - |

### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (10)*:
```solidity
File: src/Exchange.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/KangarooVault.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/LiquidityPool.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/LiquidityToken.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/PowerPerp.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/ShortCollateral.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/ShortToken.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/SynthetixAdapter.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/SystemManager.sol

2: pragma solidity ^0.8.9;

```

```solidity
File: src/utils/PauseModifier.sol

3: pragma solidity ^0.8.9;

```

### <a name="L-2"></a>[L-2] KangarooVault should initialize `systemManager`

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/KangarooVault.sol#L21

KangarooVault contract inherits from PauseModifier but doesn't initialize the `systemManager` storage variable. The contract should initialize this variable or remove the PauseModifier base contract as the pause modifier isn't used.

### <a name="L-3"></a>[L-3] LiquidityPool `withdraw` should use `safeTransfer`

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L254

The SUSD transfer on line 254 doesn't use the safe wrapper as opposed to all other calls in the contract.

### <a name="L-4"></a>[L-4] LiquidityPool `processWithdraws` overwrites `returnedAmount` for withdrawals processed in multiple steps

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L306

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L320

The `returnedAmount` field in the QueuedWithdraw struct is incorrectly overwritten when the withdraw is processed and available funds aren't enough to cover the withdrawal. As these cases are processed in multiple steps, the implementation should add the amounts instead of overwriting the value for `returnedAmount`.

```solidity
  if (susdToReturn > availableFunds) {
+     current.returnedAmount += availableFunds;
      uint256 tokensBurned = availableFunds.divWadUp(tokenPrice);
      totalQueuedWithdrawals -= tokensBurned;
      current.withdrawnTokens -= tokensBurned;
      totalFunds -= availableFunds;
      SUSD.safeTransfer(current.user, availableFunds);

      emit ProcessWithdrawalPartially(
          current.id, current.user, tokensBurned, availableFunds, current.requestedTime
          );

      return;
  } else {
      // Complete full withdrawal
+     current.returnedAmount += susdToReturn;
      totalQueuedWithdrawals -= current.withdrawnTokens;
      totalFunds -= susdToReturn;
      SUSD.safeTransfer(current.user, susdToReturn);

      emit ProcessWithdrawal(
          current.id, current.user, current.withdrawnTokens, susdToReturn, current.requestedTime
          );

      current.withdrawnTokens = 0;
  }
```

### <a name="L-5"></a>[L-5] Missing event for important parameter change

Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

*Instances (1)*:

- https://github.com/code-423n4/2023-03-polynomial/blob/main/src/LiquidityPool.sol#L650

### <a name="L-6"></a>[L-6] SystemManager initialization can be front-runned

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SystemManager.sol#L62-L82

The `init` function present in the SystemManager contract can be front-runned during the contracts bootstrapping process. Consider adding the `requiresAuth` modifier to prevent anyone from calling this function.

```solidity
  function init(
      address _pool,
      address _powerPerp,
      address _exchange,
      address _liquidityToken,
      address _shortToken,
      address _synthetixAdapter,
      address _shortCollateral
+ ) public requiresAuth {
      require(!isInitialized);
      refreshSynthetixAddresses();

      pool = ILiquidityPool(_pool);
      powerPerp = IPowerPerp(_powerPerp);
      exchange = IExchange(_exchange);
      liquidityToken = ILiquidityToken(_liquidityToken);
      shortToken = IShortToken(_shortToken);
      synthetixAdapter = ISynthetixAdapter(_synthetixAdapter);
      shortCollateral = IShortCollateral(_shortCollateral);
      isInitialized = true;
  }
```

### <a name="L-7"></a>[L-7] Synthetix market can be empty in `refreshSynthetixAddresses`

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/SystemManager.sol#L84-L87

The `refreshSynthetixAddresses` function present in the SystemManager contract refreshes the market address using the `marketForKey` function of the Synthetix `FuturesMarketManager` contract. As markets [can be removed](https://github.com/Synthetixio/synthetix/blob/v2.84.2-alpha/contracts/FuturesMarketManager.sol#L302), special care must be taken if this happens and the return value is empty.

### <a name="L-8"></a>[L-8] VaultToken `setVault` initialization can be front-runned

https://github.com/code-423n4/2023-03-polynomial/blob/main/src/VaultToken.sol#L35-L40

The function `setVault` present in the VaultToken contract can be front-runned during the Vault initialization process, which will give the caller minting capabilities over the token. 

The VaultToken can be created directly by the KangarooVault during construction time. For example, remove `setVault` and set the vault in the VaultToken constructor:

```solidity
constructor(string memory name, string memory symbol) ERC20(name, symbol, 18) {
  vault = msg.sender;
}
```

Then, KangarooVault can create the VaultToken during its own constructor:

```solidity
constructor(
    ERC20 _susd,
    IExchange _exchange,
    ILiquidityPool _pool,
    IPerpsV2Market _perpMarket,
    bytes32 _underlyingKey,
    bytes32 _name
) Auth(msg.sender, Authority(address(0x0))) {
    VAULT_TOKEN = new VaultToken(...);
    EXCHANGE = _exchange;
    LIQUIDITY_POOL = _pool;
    PERP_MARKET = _perpMarket;
    UNDERLYING_SYNTH_KEY = _underlyingKey;
    name = _name;
    SUSD = _susd;
}
```
