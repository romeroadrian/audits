## Use typed selectors in `GraphProxyAdmin` static calls

Instead of defining the literal hex values in the static calls present in the `GraphProxyAdmin` contract, the typed selectors from the `IGraphProxy` interface could be used which would improve readability, resilience and be less error prone. For example, line 33 could be changed from:

```
(bool success, bytes memory returndata) = address(_proxy).staticcall(hex"5c60da1b");
```

To:

```
(bool success, bytes memory returndata) = address(_proxy).staticcall(abi.encodeWithSelector(IGraphProxy.implementation.selector));
```

Same could be applied to 46 and 58.

## `GraphProxy` should implement `IGraphProxy`

The `GraphProxy` contract implicitly implements the functions defined in the `IGraphProxy` interface. Consider adding the explicit implementation declaration.
