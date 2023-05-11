# Report

- Non Critical Issues (4)
- Low Issues (3)

## Non Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Import declarations should import specific symbols | 10 |
| [NC-2](#NC-2) | Use named parameters for mapping type declarations | 6 |
| [NC-3](#NC-3) | Use `uint256` instead of the `uint` alias | 5 |
| [NC-4](#NC-4) | Use constants for literal config values | 27 |

### <a name="NC-1"></a>[NC-1] Import declarations should import specific symbols
Prefer import declarations that specify the symbol(s) using the form `import {SYMBOL} from "SomeContract.sol"` rather than importing the whole file

*Instances (10)*:
```solidity
File: canto-bio-protocol/src/Bio.sol

7: import "../interface/Turnstile.sol";

8: import "forge-std/Test.sol";

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

7: import "./Tray.sol";

8: import "./Utils.sol";

9: import "../interface/Turnstile.sol";

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

10: import "./Utils.sol";

11: import "../interface/Turnstile.sol";

```

```solidity
File: canto-namespace-protocol/src/Utils.sol

4: import "./Tray.sol";

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

5: import "../interface/Turnstile.sol";

6: import "../interface/ICidNFT.sol";

```

### <a name="NC-2"></a>[NC-2] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18

*Instances (6)*:
```solidity
File: canto-bio-protocol/src/Bio.sol

19:     mapping(uint256 => string) public bio;

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

43:     mapping(string => uint256) public nameToToken;

46:     mapping(uint256 => string) public tokenToName;

49:     mapping(uint256 => Tray.TileData[]) private nftCharacters;

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

67:     mapping(uint256 => TileData[TILES_PER_TRAY]) private tiles;

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

32:     mapping(uint256 => ProfilePictureData) private pfp;

```

### <a name="NC-3"></a>[NC-3] Use `uint256` instead of the `uint` alias
Prefer using the `uint256` type definition over its `uint` alias.

*Instances (5)*:
```solidity
File: canto-bio-protocol/src/Bio.sol

48:         uint lengthInBytes = bioTextBytes.length;

50:         uint lines = (lengthInBytes - 1) / 40 + 1;

56:         uint bytesOffset;

57:         for (uint i; i < lengthInBytes; ++i) {

115:             text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");

```

### <a name="NC-4"></a>[NC-4] Use constants for literal config values

*Instances (27)*:

- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L33
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L35
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L49
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L54
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L60
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L91
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L123
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L81
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L83
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L112
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L117
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L140
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L110
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L112
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L122
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L152
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L226
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L238
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L248
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L253
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L256
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L258
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L260
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L262
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L267
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L60
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L62

## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Contract files should define a locked compiler version | 4 |
| [L-2](#L-2) | Validate revenue addresses are not `address(0)` | 4 |
| [L-3](#L-3) | Namespace `burn` should delete `nftCharacters` variable  | - |

### <a name="L-1"></a>[L-1] Contract files should define a locked compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

*Instances (4)*:
```solidity
File: canto-bio-protocol/src/Bio.sol

2: pragma solidity >=0.8.0;

```

```solidity
File: canto-namespace-protocol/src/Namespace.sol

2: pragma solidity >=0.8.0;

```

```solidity
File: canto-namespace-protocol/src/Tray.sol

2: pragma solidity >=0.8.0;

```

```solidity
File: canto-pfp-protocol/src/ProfilePicture.sol

2: pragma solidity >=0.8.0;

```

### <a name="L-2"></a>[L-2] Validate revenue addresses are not `address(0)`

Check that configured addresses are not `address(0)`, else fees will be lost.

*Instances (4)*:

- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L80
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L206
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L107
- https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L220

### <a name="L-3"></a>[L-3] Namespace `burn` should delete `nftCharacters` variable

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L184-L192

The `burn` function present in the Namespace contract should also delete and clear the associated data in the `nftCharacters` variable.

```solidity
function burn(uint256 _id) external {
    address nftOwner = ownerOf(_id);
    if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
        revert CallerNotAllowedToBurn();
    string memory associatedName = tokenToName[_id];
    delete tokenToName[_id];
    delete nameToToken[associatedName];
    delete nftCharacters[_id];
    _burn(_id);
}
```
