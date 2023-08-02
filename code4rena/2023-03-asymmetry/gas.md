# `SafEth` contract:

- `derivatives[i].balance()` is called twice in the `stake` function. Consider caching the first result to avoid an extra call and read from storage.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L73-L74
  
- `derivatives[i].balance()` is called twice in the `rebalanceToWeights` function. Consider caching the first result to avoid an extra call and read from storage.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L141-L142

- In `adjustWeight`, instead of looping all derivatives to recalculate the `totalWeight`, the function can just subtract the current weight for the derivative being modified and add the new weight.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L171-L173  
  ```solidity
  function adjustWeight(
      uint256 _derivativeIndex,
      uint256 _weight
  ) external onlyOwner {
      uint256 currentWeight = weights[_derivativeIndex];
      weights[_derivativeIndex] = _weight;
      totalWeight = totalWeight - currentWeight + _weight;
      emit WeightChange(_derivativeIndex, _weight);
  }
  ```

- In the `addDerivative` function, instead of recalculating the totalWeight by looping all derivatives, the implementations can just add the weight value for the new derivative to the `totalWeight` variable.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L190-L193  
  ```solidity
  function addDerivative(
      address _contractAddress,
      uint256 _weight
  ) external onlyOwner {
      derivatives[derivativeCount] = IDerivative(_contractAddress);
      weights[derivativeCount] = _weight;
      derivativeCount++;
      totalWeight += _weight;
      emit DerivativeAdded(_contractAddress, _weight, derivativeCount);
  }
  ```

- `derivativeCount` variable is read from storage 4 times in the `addDerivative` function. Consider caching this value in a local variable.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L186  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L187  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L191  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L194  

- `minAmount` amount is read from storage while emitting the event in the `setMinAmount` function. Consider using the value from the argument of the function to save an sload.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L216

- `maxAmount` amount is read from storage while emitting the event in the `setMaxAmount` function. Consider using the value from the argument of the function to save an sload.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L225
  
- `pauseStaking` value is read from storage while emitting the event in the `setPauseStaking` function. Consider using the value from the argument of the function to save an sload.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L234
  
- `pauseUnstaking` value is read from storage while emitting the event in the `setPauseUnstaking` function. Consider using the value from the argument of the function to save an sload.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L243

# `Reth` contract

- Consider using an infinite token approval to the Uniswap pool instead of approving each individual token transfer for each swap involved in the calls to the `deposit` function.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L90
  
- The `ethPerDerivative` function multiplies and divides the result from `poolPrice` by the same amount (10 ** 18). Consider returning just the value from `poolPrice` to save gas.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L215

- Uniswap V3 pool addresses are deterministic and can be computed locally instead of making an external call to the factory. See [PoolAddress.computeAddress](https://github.com/Uniswap/v3-periphery/blob/main/contracts/libraries/PoolAddress.sol#L33).  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L238


# `SfrxEth` contract

- In the `withdraw` function, use the return value of the call to `redeem` to know how much frxETH was redeemed instead of making a call to fetch the contract balance.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L61-L68
  
- Consider using an infinite token approval to the Curve pool instead of approving each individual token transfer for each swap involved in the calls to the `withdraw` function.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L69-L72

- In the `deposit` function, use the return value of the call to `submitAndDeposit` to know how much sfrxETH was deposited instead of calculating it as the difference of balances pre/post execution.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L98-L105  
  ```solidity
  function deposit() external payable onlyOwner returns (uint256) {
      IFrxETHMinter frxETHMinterContract = IFrxETHMinter(
          FRX_ETH_MINTER_ADDRESS
      );
      return frxETHMinterContract.submitAndDeposit{value: msg.value}(address(this));
  }
  ```
  
- In the `withdraw` function, use the return value of the call to `exchange` to know how much ETH was swapped.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L77

# `WstEth` contract

- Consider using an infinite token approval to the Curve pool instead of approving each individual token transfer for each swap involved in the calls to the `withdraw` function.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L59

- In the `withdraw` function, use the return value of the call to `unwrap` to know how much stETH was unwrapped instead of making a call to fetch the contract balance.  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L57  
  ```solidity
  function withdraw(uint256 _amount) external onlyOwner {
      uint256 stEthBal = IWStETH(WST_ETH).unwrap(_amount);
      IERC20(STETH_TOKEN).approve(LIDO_CRV_POOL, stEthBal);
      uint256 minOut = (stEthBal * (10 ** 18 - maxSlippage)) / 10 ** 18;
      IStEthEthPool(LIDO_CRV_POOL).exchange(1, 0, stEthBal, minOut);
      // solhint-disable-next-line
      (bool sent, ) = address(msg.sender).call{value: address(this).balance}(
          ""
      );
      require(sent, "Failed to send Ether");
  }
  ```

# General

- There are several places throughout the code that could benefit from using a gas efficient math library such as [solmate FixedPointMathLib](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol).  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L81  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L92  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L98  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L115  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol#L149  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L173  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/Reth.sol#L241  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L74  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/SfrxEth.sol#L115  
  https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/derivatives/WstEth.sol#L60  
