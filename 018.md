0x0

medium

# Fixed Decimal Fee Calculation

## Summary

Top ups are being processed with the assumption that the token always has 18 decimals.

## Vulnerability Detail

[`HardenedTopupProxy._processTopup`](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L296)

The fee is calculated with the assumption that the token has 18 decimals. The calculation is incorrect for tokens that do not use 18 decimals.

## Impact

Should an asset be added to the system that does not use 18 decimals, such as Tether, the top up fee will be incorrect.

## Code Snippet

```solidity
uint256 feeAmount = _amount.mul(topupFee).div(1e18);
```

## Tool used

Manual Review

## Recommendation

Consider whether the addition of tokens with non-standard number of decimals is desirable in the future. If so, refactor this function to accommodate assets that do not use 18 decimals.
