WATCHPUG

high

# The value of `to` parameter in `_bridgeTxData` can be malicious

## Summary

The attacker can maliciously set the `to` address in the `_bridgeTxData` to their own address.

## Vulnerability Detail

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L315-L326

When `_bridgeType == 0`, there is no data validation to ensure the `to` address in the `callData` is set as expected to the `cardPartnerAddress`.

An attacker can set the `to` to their own address on L1 and retrieve the funds back on L1.

In the current implementation, both the 2 events (`CardTopup` and `BridgeTx`) would be emitted in such a way that they are indistinguishable from another regular top-up transaction.

Even if the backend checks for the status of the bridge transaction, it will be a successful one.

## Impact

The system may be deceived into believing that the bridge is authentic and successful and top up the funds to the attacker's card.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L395-L444

https://polygonscan.com/address/0x1c6aE197fF4BF7BA96c66C5FD64Cb22450aF9cC8#code

## Tool used

Manual Review

## Recommendation

Consider extracting the `to` address from the calldata and validate it when `_bridgeType == 0`.