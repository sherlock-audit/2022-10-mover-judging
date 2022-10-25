berndartmueller

medium

# The Synapse bridge integration does not validate the low-level call function parameter and can lead to incorrect bridging

## Summary

Allowing arbitrary `callData` to be passed to the Synapse bridge contract call makes it possible to call any function (accidentally or due to a bug in the frontend) on the target contract. This can lead to incorrect bridging of tokens.

## Vulnerability Detail

Tokens can be either bridged via the _Synapse bridge_ or the _Across bridge_. The Across bridge is called through a well-defined interface. The Synapse bridge, however, allows arbitrary `callData` to be passed to the `call` function. In case of accidentally using the wrong Synapse contract functions (e.g. a view-only function), the call will succeed but the tokens will not be bridged. As the necessary top-up events are emitted, the off-chain nodes will consider the bridging to be successful. This will lead to issues with the top-up or accounting and requires manual intervention to fix the issue.

## Impact

The Synapse bridge contract can be called with a `callData` parameter that includes a call to a function which **does not** bridge tokens, instead, it could be a call to a view-only function (or, if the Synapse bridge contract would have a `fallback` function, empty calldata would suffice to have a successful call).

The call would succeed, and the transaction succeeds with the necessary `CardTopup` event emitted, but the tokens will not be bridged. This would then require manual intervention to fix the issue and return token funds to the user.

## Code Snippet

[HardenedTopupProxy.sol#L421](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L421)

```solidity
function _bridgeAssetDirect(uint256 _amount, uint256 _bridgeType, bytes memory _bridgeTxData) internal {
    [...]

    if (_bridgeType == 0)
    {
        // Synapse bridge call data is retrieved by performing a call by the application
        // to bridge SDK and is not transformed by this contract
        bytes memory callData = _bridgeTxData.slice(20, _bridgeTxData.length - 20);
        (bool success, ) = targetAddress.call(callData); // @audit-info `callData` is not validated and can contain any data
        require(success, "BRIDGE_CALL_FAILED");
    } else if (_bridgeType == 1) {

    [...]
}
```

## Tool Used

Manual Review

## Recommendation

Consider not allowing arbitrary `callData` to be passed to the Synapse bridge and instead only allowing the `callData` to be constructed by the contract itself and provide necessary parameters.
