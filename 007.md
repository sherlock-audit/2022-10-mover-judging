csanuragjain

medium

# No provision to pause ExchangeProxy

## Summary
It was observed that unlike HardenedTopupProxy, ExchangeProxy does not have a pause functionality and executeSwap can run even in worst scenarios

## Vulnerability Detail
1. Observe that ExchangeProxy contract does not have pausing capability allowing anyone to call executeSwapDirect even when owner want to pause the functionality due to an exploit

## Impact
No way to pause the swapping

## Code Snippet
https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L131

## Tool used
Manual Review

## Recommendation
Add pausing functionality and allow admin to pause executeSwapDirect/executeSwap whenever required