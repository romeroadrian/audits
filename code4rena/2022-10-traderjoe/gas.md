## Checks in the `addLiquidity` function of the router can be moved up in the stack

The deadline check (line 658), array length check (line 662) and bin id bound check (line 665) can be moved up in the stack (since they don't depend on anything else other the function parameters) to early exit in case any of the conditions isn't met and save some gas.
