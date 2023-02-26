# Storage variables packing

The storage variable `isOpen` can be changed to a `bool` type to use 1 byte and be packed in a single storage slot with one of the addresses that follow next.

Similarly, the blockRange variable can be reduced in size to a uint96 (which will still allow for a max value of `79228162514264337593543950335`) to be packed with one of the storage addresses and fit a single storage slot.

# Change the `weth` variable to be immutable

The address for the WETH contract can be changed to be immutable (or constant) so it's fixed in the bytecode and isn't loaded from storage state, since it's known at deploy time and shouldn't be subject to change. 
