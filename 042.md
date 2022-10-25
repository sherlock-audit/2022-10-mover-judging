berndartmueller

medium

# The time-dependent signature check is not safe

## Summary

The `HardenedTopupProxy.CardTopupTrusted` function is called by a trusted off-chain executor and uses a time-dependent signature check. This is unsafe as the signature can be (accidentally) reused within the allowed `allowanceSignatureTimespan` timespan.

## Vulnerability Detail

The `HardenedTopupProxy.CardTopupTrusted` function is called by a trusted off-chain executor and uses a time-dependent signature check by incorporating a timestamp `_timestamp` in the signed signature. To prevent replay attacks, the `_timestamp` is checked to be within the allowed `allowanceSignatureTimespan` timespan. However, it is in the possible realm of the off-chain trusted executor to retry the call (due to various reasons) with the same parameters (given that the token spending allowance is set as well, or, the allowance is already set high enough to cover multiple topups with the same amount - as long as `allowanceTreshold` is set accordingly).

## Impact

The trusted off-chain executor can use the same signature multiple times within the allowed `allowanceSignatureTimespan` timespan. This can lead to repeated top-ups for the receiver `_receiverHash`.

## Code Snippet

[HardenedTopupProxy.sol#L1041](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L1041)

```solidity
function CardTopupTrusted(address _token, uint256 _amount, uint256 _timestamp, bytes calldata _signature, uint256 _expectedMinimumReceived, bytes memory _convertData, uint256 _bridgeType, bytes memory _bridgeTxData, bytes32 _receiverHash) public {
    // reconstruct message from the data used for topup to ensure it matches provided information
    bytes32 message = constructMsg(keccak256(abi.encodePacked(msg.sender)), _token, _amount, _timestamp);

    // recover signer which must be trusted executor EOA
    address signer = recoverSigner(message, _signature);
    require(hasRole(TRUSTED_EXETUTOR_ROLE, signer), "wrong signature");

    // if signature is old, don't accept it to avoid reuse and ensure approval was fresh
    // trusted executor won't produce messages with timestamps in the future
    require(block.timestamp - _timestamp < allowanceSignatureTimespan, "old sig");

    // check current allowance, regardless of the signed message vailidity
    checkAllowance(_token, _amount);

    // allowance checks/enforcing complete, perform actual topup (this flow is further unified across 3 possible topup public methods)
    _processTopup(msg.sender, _token, _amount, _expectedMinimumReceived, _convertData, _bridgeType, _bridgeTxData, _receiverHash);
}
```

## Tool Used

Manual Review

## Recommendation

Consider using a nonce to prevent replay attacks. This can be done by adding a nonce to the signed message instead of the timestamp `_timestamp`. The nonce can be stored and incremented on every `CardTopupTrusted` function call. This way the same signature can not be used twice.
