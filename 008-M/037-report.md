berndartmueller

high

# Previous yield distributor can drain collected fees

## Summary

The yield distributor has the maximum token spending allowance for the `HardenedTopupProxy.sol` and `ExchangeProxy` contracts. However, when setting a new and different yield distributor, the old yield distributor still has the maximum spending allowance and can drain all collected fees.

## Vulnerability Detail

When setting a new yield distributor with the `setYieldDistributor` function in both the `HardenedTopupProxy.sol` and `ExchangeProxy` contracts, the yield distributor address will have the ERC-20 token with the address `_tokenAddress` approved with the maximum allowance `ALLOWANCE_SIZE = type(uint256).max` as a spender.

If the contract admin changes the current yield distributor to a different address, the previous address still has the spending allowance. This means that the previous yield distributor can spend the tokens from both contracts.

## Impact

A previously set yield distributor continues to have the ERC-20 token spending allowance and can drain the collected fees.

## Code Snippet

[ExchangeProxy.setYieldDistributor](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L228)

```solidity
function setYieldDistributor(address _tokenAddress, address _distributorAddress) public onlyAdmin {
    yieldDistributorAddress = _distributorAddress;
    // only yield to be redistributed should be present on this contract in baseAsset (or other tokens if swap fees)
    // so no access to lp tokens for the funds invested
    resetAllowanceIfNeeded(IERC20(_tokenAddress), _distributorAddress, ALLOWANCE_SIZE);
}
```

[HardenedTopupProxy.setYieldDistributor](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L196)

```solidity
function setYieldDistributor(address _tokenAddress, address _distributorAddress) public onlyAdmin {
    yieldDistributorAddress = _distributorAddress;
    // only yield to be redistributed should be present on this contract balance in baseAsset
    resetAllowanceIfNeeded(IERC20Upgradeable(_tokenAddress), _distributorAddress, ALLOWANCE_SIZE);
}
```

## Tool Used

Manual Review

## Recommendation

Consider removing the call to the `resetAllowanceIfNeeded` function within the two `setYieldDistributor` functions and only let the current yield distributor claim fees via the provided `claimFees` function.
