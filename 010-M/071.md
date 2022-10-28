Dravee

medium

# `minAmount` can be above `maxAmount`

## Summary
Illogical state: `minAmount` can be above `maxAmount`

## Vulnerability Detail
Be it by mistake or with ill intents, it shouldn't be possible to set `minAmount` above `maxAmount` or `maxAmount` below `minAmount`

## Impact
Depends on the calling contracts' use-cases. Potential DOS.

## Code Snippet
https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L218-L220

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L222-L224

## Tool used

Manual Review

## Recommendation
In those setters, consider adding those checks (a strict inequality might be more relevant):
```diff
File: HardenedTopupProxy.sol
218:     function setMinAmount(uint256 _minAmount) public onlyAdmin {
+ 219:         require(_minAmount <= maxAmount);
219:         minAmount = _minAmount;
220:     }
221: 
222:     function setMaxAmount(uint256 _maxAmount) public onlyAdmin {
+ 223:         require(_maxAmount >= minAmount);
223:         maxAmount = _maxAmount;
224:     }
```
