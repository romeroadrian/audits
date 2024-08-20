# Biconomy Nexus Audit - CodeHawks

Contest: https://codehawks.cyfrin.io/c/2024-07-biconomy

## High

- [H-01 Fallback handler is missing authorization control](./H-01.md)
- [H-02 Module type argument is malleable in Module Enable Mode flow](./H-02.md)
- [H-03 ETH is not forwarded to factory call in BiconomyMetaFactory](./H-03.md)
- [H-04 Bootstrap functions setup the registry after module installation, bypassing the validation checks](./H-04.md)

## Medium

- [M-01 Typehash for ModuleEnableMode struct is incorrect](./M-01.md)
- [M-02 solady handler prevents module extensibility for ERC721 and ERC1155 callbacks](./M-02.md)
- [M-03 Missing support for ERC-165](./M-03.md)
- [M-04 Missing call type validation for fallback handlers](./M-04.md)
- [M-05 Missing callvalue handling in `executeUserOp()`](./M-05.md)
- [M-06 Hooks are not triggered for `executeUserOp()`](./M-06.md)
- [M-07 Module uninstallation is vulnerable to denial of service attacks](./M-07.md)
- [M-08 Calldata length is not validated in `fallback()`](./M-08.md)
- [M-09 RegistryFactory should validate that there are no duplicate attesters](./M-09.md)
- [M-10 Account creation in RegistryFactory doesn't validate the initialization data targets a known bootstrapper](./M-10.md)
