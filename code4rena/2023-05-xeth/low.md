# Report

- Low Issues (11)
- Informational Issues (2)
- Non-Critical Issues (3)

## Low Issues

| |Issue|
|-|:-|
| [L-1](#L-1) | Wrong function declaration in IBaseRewardPool interface |
| [L-2](#L-2) | Incorrect function declaration in ICurveFactory interface |
| [L-3](#L-3) | `grantRole` operation in `setAMO` function may incorrectly check role access |
| [L-4](#L-4) | Potential accidental inclusion of ERC20Burnable in xETH token |
| [L-5](#L-5) | `addLockedFunds` should check `dripRatePerBlock` is not zero |
| [L-6](#L-6) | Use Ownable2Step instead of Ownable for access control |
| [L-7](#L-7) | `recoverToken` function should restrict which tokens are allowed to be recovered |
| [L-8](#L-8) | `setOperator` and `setRewardsRecipient` don't check for `address(0)` |
| [L-9](#L-9) | Initialize `rebalanceUpCap` and `rebalanceDownCap` in AMO constructor |
| [L-10](#L-10) | Validate argument in `setRebalanceUpThreshold` and `setRebalanceDownThreshold` |
| [L-11](#L-11) | `rebalanceUpCap` and `rebalanceDownCap` operate on different types of value |

### <a name="L-1"></a>[L-1] Wrong function declaration in IBaseRewardPool interface

The `withdraw` function declaration is missing the `bool` return value.

https://github.com/code-423n4/2023-05-xeth/blob/main/src/interfaces/IBaseRewardPool.sol#L14

```solidity
function withdraw(uint256 amount, bool claim) external;
```

### <a name="L-2"></a>[L-2] Incorrect function declaration in ICurveFactory interface

The `deploy_pool` function doesn't exist in the Curve Factory contract (see https://github.com/curvefi/curve-factory/blob/master/contracts/Factory.vy).

https://github.com/code-423n4/2023-05-xeth/blob/main/src/interfaces/ICurveFactory.sol#L5-L18

```solidity
function deploy_pool(
    string memory name,
    string memory symbol,
    address[] memory coins,
    uint256 A,
    uint256 mid_fee,
    uint256 out_fee,
    uint256 allowed_extra_profit,
    uint256 fee_gamma,
    uint256 adjustment_step,
    uint256 admin_fee,
    uint256 ma_half_time,
    uint256 initial_price
) external returns (address);
```

### <a name="L-3"></a>[L-3] `grantRole` operation in `setAMO` function may incorrectly check role access

https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L54

Instead of using the internal `_grantRole` function, the `setAMO` function present in the xETH is using the `grantRole` function of the OpenZeppelin AccessControl library, which includes a validation that the caller has the admin role for the role being granted (`MINTER_ROLE` in this case):

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol#L145-L147

```solidity
function grantRole(bytes32 role, address account) public virtual override onlyRole(getRoleAdmin(role)) {
    _grantRole(role, account);
}
```

This should be ok as long as the admin role for `MINTER_ROLE` is not defined. Currently, the `setAMO` function will be called by the `DEFAULT_ADMIN_ROLE` which is the default admin role for the `MINTER_ROLE` role. But if the role admin for the `MINTER_ROLE` is defined, then the default admin role will not be the admin role for the `MINTER_ROLE` and the operation will be reverted.

The recommendation is to change `grantRole` for the internal variant `_grantRole`.

### <a name="L-4"></a>[L-4] Potential accidental inclusion of ERC20Burnable in xETH token

https://github.com/code-423n4/2023-05-xeth/blob/main/src/xETH.sol#L9

The xETH token inherits from ERC20Burnable, which defines public methods to allow burning of tokens.

Presumably this was added accidentally as a mistake in order to implement the `burnShares()` function in xETH. The internal function `_burn` is already provided in the base implementation of ERC20 (OpenZepplin library), and doesn't need any extra implementation, which may increase the attack surface of the protocol.

The recommendation is to remove ERC20Burnable from the inheritance list of xETH.

### <a name="L-5"></a>[L-5] `addLockedFunds` should check `dripRatePerBlock` is not zero

https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L146-L148

The `addLockedFunds` function present in the wxETH contract should check that `dripRatePerBlock > 0`.

### <a name="L-6"></a>[L-6] Use Ownable2Step instead of Ownable for access control

- https://github.com/code-423n4/2023-05-xeth/blob/main/src/wxETH.sol#L9
- https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L11

Use the [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) variant of the Ownable contract to better safeguard against accidental transfers of access control.

### <a name="L-7"></a>[L-7] `recoverToken` function should restrict which tokens are allowed to be recovered

https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L101-L109

The CVXStaker contract manages different tokens as part of its natural intended behavior, notably these are the Curve LP tokens, Convex tokens, and the different reward tokens associated with the Convex staking pool.

The `recoverToken` implementation can arbitrarily transfer any ERC20 token, which may also be used to transfer these tokens.

The recommendation is to restrict these tokens, which are an important part of the process around the contract, and should not be "recovered" as part of an accidental side effect.

### <a name="L-8"></a>[L-8] `setOperator` and `setRewardsRecipient` don't check for `address(0)`

- https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L78
- https://github.com/code-423n4/2023-05-xeth/blob/main/src/CVXStaker.sol#L89

Check that the arguments are not `address(0)`.

### <a name="L-9"></a>[L-9] Initialize `rebalanceUpCap` and `rebalanceDownCap` in AMO constructor

https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L175

Initialize these two variables to comply with the invariant of these values being different from zero, which is checked in their respectives setters.

### <a name="L-10"></a>[L-10] Validate argument in `setRebalanceUpThreshold` and `setRebalanceDownThreshold`

- https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L495
- https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L512

Validate that the thresholds are within the 0-1e18 range in `setRebalanceUpThreshold` and `setRebalanceDownThreshold` function of the AMO contract.

### <a name="L-11"></a>[L-11] `rebalanceUpCap` and `rebalanceDownCap` operate on different types of value

Caps operate on different types of value. `rebalanceUpCap` is checked against LP tokens:

https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L252

```solidity
if (quote.lpBurn > rebalanceUpCap) revert RebalanceUpCapExceeded();
```

While `rebalanceDownCap` is checked against xETH tokens:

https://github.com/code-423n4/2023-05-xeth/blob/main/src/AMO2.sol#L299

```solidity
if (quote.xETHAmount > rebalanceDownCap)
    revert RebalanceDownCapExceeded();
```

## Informational Issues

| |Issue|
|-|:-|
| [INFO-1](#INFO-1) | Type of Curve pool is an important and sensible parameter |
| [INFO-2](#INFO-2) | Several centralization risks introduce multiple points of failure |

### <a name="INFO-1"></a>[INFO-1] Type of Curve pool is an important and sensible parameter

Protocol owners will deploy a Curve pool as an essential piece of the protocol. The current intention is to use a Curve for the pair xETH/stETH.

The protocol team has asserted that they will be using a plain pool, "balances" type, in which none of the coins are native ETH.

There are two issues that shouldn't be a problem as long as these conditions are maintained:

1. stETH being a rebasing token needs the Curve pool to be of "balances" type. Metapools are not supported either.
2. As the protocol uses Curve's `get_virtual_price()` (in the AMO contract) it can potentially be vulnerable to the read-only reentrancy attacked described in the [following article](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/). This attack is not possible as long as none of the coins is ETH, which is the main driver to trigger the reentrancy.

If conditions are changed, and the type of Curve pool is changed, these two potential issues could be manifested as vulnerabilities in the protocol. 

### <a name="INFO-2"></a>[INFO-2] Several centralization risks introduce multiple points of failure

Although much care has been taken to contain and bound the accessible functionality of the defender role, there are multiple centralization risks around the protocol that imply a lot of trust in owners or controllers of the protocol.

Here is a non-exhaustive list of the different places that may introduce potential failures due to centralization:

- xETH can be paused, freezing all operations.
- AMO can be changed in xETH, which grants arbitrary minting capabilities to the new account.
- Minting and burning is also accessible to the `DEFAULT_ADMIN_ROLE`.
- Configuration for wxETH rewards can be arbitrarily changed.
- Rewards for wxETH are handled manually via off-chain processes (claim revenue and call to add locked funds).
- Rebalance operations have cooldown, but cooldown can be changed to zero.
- Rebalance can be neutralized by setting zero caps.
- Rebalance thresholds can be arbitrarily changed to any amount.
- Remove liquidity functions in AMO contract can be used to empty the Curve pool.
- CXVStaker can be changed in AMO contract (sensible because this contract controls Curve LPs).
- LP tokens in CVXStaker can be withdrawn and sent to an arbitrary account (`withdrawAndUnwrap`).

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 20 |
| [NC-2](#NC-2) | Use `uint256` instead of the `uint` alias | 2 |
| [NC-3](#NC-3) | The `accrueDrip()` function can use `_accrueDrip()` to avoid the empty block implementation | - |

### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (20)*:
```solidity
File: src/AMO2.sol

4: import "@openzeppelin-contracts/token/ERC20/IERC20.sol";

5: import "@openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

6: import "@openzeppelin-contracts/access/AccessControl.sol";

7: import "./interfaces/ICurvePool.sol";

```

```solidity
File: src/CVXStaker.sol

4: import "@openzeppelin-contracts/token/ERC20/IERC20.sol";

5: import "@openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

6: import "@openzeppelin-contracts/access/Ownable.sol";

7: import "./interfaces/ICurvePool.sol";

8: import "./interfaces/ICVXBooster.sol";

9: import "./interfaces/IBaseRewardPool.sol";

```

```solidity
File: src/interfaces/IXETH.sol

4: import "@openzeppelin-contracts/token/ERC20/IERC20.sol";

```

```solidity
File: src/wxETH.sol

3: import "@openzeppelin-contracts/token/ERC20/ERC20.sol";

4: import "@openzeppelin-contracts/token/ERC20/IERC20.sol";

5: import "@openzeppelin-contracts/token/ERC20/utils/SafeERC20.sol";

6: import "@openzeppelin-contracts/access/Ownable.sol";

7: import "solmate/utils/FixedPointMathLib.sol";

```

```solidity
File: src/xETH.sol

4: import "@openzeppelin-contracts/token/ERC20/ERC20.sol";

5: import "@openzeppelin-contracts/token/ERC20/extensions/ERC20Burnable.sol";

6: import "@openzeppelin-contracts/security/Pausable.sol";

7: import "@openzeppelin-contracts/access/AccessControl.sol";

```

### <a name="NC-2"></a>[NC-2] Use `uint256` instead of the `uint` alias
Prefer using the `uint256` type definition over its `uint` alias.

*Instances (2)*:
```solidity
File: src/CVXStaker.sol

195:             for (uint i = 0; i < rewardTokens.length; i++) {

```

```solidity
File: src/wxETH.sol

16:     event UpdateDripRate(uint oldDripRatePerBlock, uint256 newDripRatePerBlock);

```

### <a name="NC-3"></a>[NC-3] The `accrueDrip()` function can use `_accrueDrip()` to avoid the empty block implementation

The implementation of the `accrueDrip()` function can be changed to use `_accrueDrip()` instead of the `drip` modifier to avoid the empty block implementation.

```solidity
function accrueDrip() external {
  _accrueDrip();
}
```
