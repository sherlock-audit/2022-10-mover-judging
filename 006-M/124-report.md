WATCHPUG

medium

# Slippage tolerance for Synapse should not be specified as constant values of `0.95`, `0.91`

## Summary

Constant values are being used as the slippage tolerance for Synapse, which may result in the user's loss due to improper slippage control.

## Vulnerability Detail

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L296-L385

`minMint` is hardcoded as `0.95`, and `minDy` is hardcoded as `0.91`, which is expected to be specified and controlled by the user themselves from the frontend.

While a swap before the bridge call makes it hard to have the specific values, it's still possible to pass the slippage tolerance ratio from the frontend and do the calculation onchain.

## Impact

Hardcoded slippage tolerance may result in the user's loss due to improper slippage control.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L296-L385

## Tool used

Manual Review

## Recommendation

Consider allowing the frontend to specify the slippage tolerance ratio.
