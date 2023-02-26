## Duplicate getters for unstructured storage access

Most of the contracts have an internal function to sload a storage variable (due to the usage of unstructured storage), and also have an external function which does exactly the same. For example in `HolographBridge`, there's `getHolograph()` and `_holograph()` which have the same code. The same happens with most of the unstructured storage variables in most of the contracts in scope. Consider using a single public function.

## Duplicate calls in `isOwner()` function of `PA1D` contract

This function checks if the sender matches the owner and admin, and then calls each getter a second time by executing an external call. 

## Define constant for basis points in `PA1D` contract

There are several references to `10000` which is the basis point for the bps calculation. Consider defining a constant. Occurrences are in lines 395, 438, 477, 551, 553, 641, 643.

## Change `_validatePayoutRequestor` to be a modifier in `PA1D` contract

The `_validatePayoutRequestor()` function present in the `PA1D` contract works as a guard for the payout function. Consider moving it to a modifier.

## No need to define parameter `receiver` as payable in `setRoyalties` function of `PA1D` contract

The `receiver` parameter is marked as payable though it isn't needed.

## `getFees` and `getRoyalties` have the same duplicated code in `PA1D` contract

The body of both functions is exactly the same. Consider extracting duplicated code to a private function.

## The `init` function in the `LayerZeroModule` contract doesn't initialize `lZEndpoint`

Though there's an admin setter to set this value, consider initializing the `lZEndpoint` variable (`_lZEndpointSlot` slot) in the initializer so that it's fully set up at construction time. Note that this is a crucial part of the bridging process, and failure to correctly initialize this variable will make the whole process fail.

## Unnecessary usage of low level assembly code in function `lzReceive` of `LayerZeroModule` contract

This function relies on assembly to simply do a couple of checks and revert with an error in case the conditions are not met. Consider using solidity, which will improve readability and will be less error prone.

## Don't define `receive` and `fallback` functions for the default behavior

Several contracts define the `receive` and/or `fallback` functions with a single instruction to revert. This matches the default behavior taken by solidity if those functions aren't defined.

Contracts where this is present: `LayerZeroModule`, `HolographBridge`, `HolographFactory`, `HolographOperator`.

## `if (condition) { return true } else { return false }` code construction should just return the condition

Constructions that test a condition and return true or false based on that condition should directly return the condition. 

Occurrences:

- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC721.sol#L298
- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L289
- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L755

# Unnecessary functions marked as payable

There's no indication that these should receive a payment though they are marked as payable.

- `approve(address to, uint256 tokenId)` in `HolographERC721` contract.
- `safeTransferFrom(address from, address to, uint256 tokenId)` in `HolographERC721` contract.
- `safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data)` in `HolographERC721` contract.
- `transfer(address to, uint256 tokenId)` in `HolographERC721` contract.
- `transferFrom(address from, address to, uint256 tokenId)` in `HolographERC721` contract.
- `transferFrom(address from, address to, uint256 tokenId, bytes memory data)` in `HolographERC721` contract.

## Use solidity `interfaceId` to test for ERC165 interface support

In `HolographERC721` contract line 465 and 466 the code uses the selectors of ERC165 and ERC721TokenReceiver to test for ERC165 interface support. This works because those interfaces contain a single function definition. Consider using instead `type(ERC165).interfaceId` and `type(ERC721TokenReceiver).interfaceId`.

In `HolographERC20` contract line 644 use `type(ERC165).interfaceId` and in line 647 use `type(ERC20Safer).interfaceId` instead of the magic constant "0x534f5876".

## Simplify `_isContract` implementation

Several contracts in the codebase implement a `_isContract` function to check if an address contains code through the `extcodehash` opcode using low level assembly. Consider replacing this with the simpler expression `contractAddress.code.length > 0`, which doesn't rely on assembly code.

Occurrences: `HolographERC721`, `HolographERC20`, `HolographOperator`, `HolographFactory`.

## Naming convention for private functions

The `SourceERC721()` and `SourceERC20()` function present in the `HolographERC721` and `HolographERC20` contracts are private functions and don't follow the naming convention used mostly throughout the code where these type of functions are named starting with an underscore and a lowercase letter.

## Define literal magic constants as named constants

- `_royalties` function in `HolographERC721` contract, value `0x0000000000000000000000000000000000000000000000000000000050413144`.
- `permit` function in `HolographERC20` contract. Magic value `0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9` can be defined as a constant calculated using `keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)")`.

## No need to define low level literals as 32 bytes left padded with zeros

There are several places throughout the code base that define literals in low level code by left padding it with zeros to make those be 32 bytes long. This is unnecessary and damages readability. For example, `HolographERC20` contract line 222 contains `0x0000000000000000000000000000000000000000000000000000000000000001` just to express a `1`.

## Incorrect EIP712 initialization in `HolographERC20` contract

The initialization of the `HolographERC20` contract calls the `_eip712_init` function with a `domainSeperator` parameter as the first argument. Aside from the spelling mistake, this argument should correspond to the name field of the EIP712 construct, the domain separator is the output of that function that is handled internally.

## Unnecessary use of unchecked math in `increaseAllowance` in `HolographERC20` contract

The code uses unchecked math to calculate the new allowance in line 424, just to check the overflow in line 427 (this line is also unnecessarily wrapped in an unchecked block since this is just a comparison). Safe math could be directly used here to simplify the construction.

```
uint256 newAllowance = currentAllowance + addedValue;
```

## Nested if constructions can be defined using a single condition

An if construction nested in another if construction could have both guards combined to form a single if statement.

- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L526 
- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L596 

## Check for same length array argument functions

Function that operate simultaneously on more than one array should sanity check that those arrays have the same length.

Occurrences:

- https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/enforcer/HolographERC20.sol#L563

## Rename `msgSender()` function in `ERC20H` and `ERC721H` contracts 

The function named `msgSender()` could be inadvertently confused with an utility function that returns `msg.sender`. For example, OpenZeppelin's `Context` contract defines a very similar function named `_msgSender()`, this is a highly popular library present in most solidity codebases.

Consider renaming this function to something like `holographerMsgSender()`.

## Improve low level variable override in `jobEstimator` function of the `HolographOperator` contract

The `doNotRevert` variable is set to `0` by executing a low level call to `mstore8(0xE3, 0x00)`. This should be ok, since that byte corresponds to the last byte of the word that represents that boolean parameter. However, consider using `mstore(0xC4, 0)` instead since it better aligns with the intention of overriding the `doNotRevert` argument, which is the seventh argument in the payload considering the 4 byte selector (`0x04 + 6 * 0x20 = C4`).

## Incorrect variable size when decoding job in `getJobDetails` of the `HolographOperator` contract

The variable `startTimestamp` should be defined as a `uint80` instead of `uint64` to align it with how it is initially packed (it occupies bits 16 to 96, `96-16=80`).

## Consider using EIP712 to sign structured data in `deployHolographableContract` of the `HolographFactory` contract

This function calculates a hash over the `DeploymentConfig` hash to verify the submitted signature. This is an ideal case to use the more user friendly EIP712 hashing and signing.

## `supportsInterface` should return true for the EIP165 interface ID in `ERC20H` and `ERC721H`

Both contracts define the `supportsInterface(bytes4)` function and return just `false` for all cases. According to the EIP-165 specification, this function should return `true` when called with the value `0x01ffc9a7` (the interface id for the `IERC165` interface).

Change the implementation to:

```
function supportsInterface(bytes4 interfaceId) external pure returns (bool) {
  return interfaceId == type(IERC165).interfaceId;
}
```
