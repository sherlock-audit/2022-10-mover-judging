rvierdiiev

medium

# If cardTopupToken == ETH_TOKEN_ADDRESS then it's not possible to top up with native token

## Summary
If `cardTopupToken == ETH_TOKEN_ADDRESS` then users will not be able to top up using native token payment, because cardTopupToken is expected to be ERC20 token only.
## Vulnerability Detail
In `HardenedTopupProxy._processTopup` there is a [check](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L315) that checks if user provided same token as `cardTopupToken`. In this case it will try to [send](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L317) funds to `HardenedTopupProxy` and then bridge it.

However if `cardTopupToken == ETH_TOKEN_ADDRESS` then the transfer will fail as ETH_TOKEN_ADDRESS is used for native token payment.

Why it's possible that `cardTopupToken == ETH_TOKEN_ADDRESS`? Because there is no checks for what `cardTopupToken` address can be. So it's possible for admin to [set](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L214-L216) it to ETH_TOKEN_ADDRESS.
## Impact
Bridging will be not possible for native token payment.
## Code Snippet
```solidity
        if (_token == cardTopupToken) {
            // beneficiary is msg.sender (perform static check)
            IERC20Upgradeable(_token).safeTransferFrom(_beneficiary, address(this), _amount);

            uint256 feeAmount = _amount.mul(topupFee).div(1e18);

            // bridge from _beneficiary to card L1 relay
            _bridgeAssetDirect(_amount.sub(feeAmount), _bridgeType, _bridgeTxData);

            emit CardTopup(_beneficiary, _token, _amount, _amount.sub(feeAmount), _receiverHash);
            return;
        }
```
## Tool used

Manual Review

## Recommendation
Add check to the setter method, that provided token can't be `ETH_TOKEN_ADDRESS`.