WATCHPUG

medium

# Lack of sanity checks in the setter functions can result in malfunctions

## Summary

There are many settings that can be changed by the admin at any time, but there are not enough sanity checks to ensure it won't break the system.

## Vulnerability Detail

For instance:

- Set `topupFee` or `exchangeFee` to `> 1e18` will result in underflow when deducting the fees:
    
    https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L191-L194

    https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L319-L322

- Set `minAmount` > `maxAmount` will make it impossible to topup;

    https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L396-L397

## Impact

Malfunctioning of multiple major features when improper values are used.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L199-L229

## Tool used

Manual Review

## Recommendation

Consider adding the missing sanity checks.