JohnSmith

medium

# No Storage Gap for Upgradeable Contract Might Lead to Storage Slot Collision

## Summary
For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments” (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. 
## Vulnerability Detail
Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract.
Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

The storage gap is essential for upgradeable contract because “It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments”. Refer to the bottom part of this article:

https://docs.openzeppelin.com/contracts/3.x/upgradeable
## Impact
   If the contract inheriting the base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.
   This could have unintended and very serious consequences to the child contracts, potentially causing loss of funds or cause the contract to malfunction completely.
## Code Snippet
`HardenedTopupProxy` is an upgradable contract which extends  `AccessControlUpgradeable` and `SafeAllowanceResetUpgradeable`
```solidity
contracts/HardenedTopupProxy.sol
68: contract HardenedTopupProxy is AccessControlUpgradeable, SafeAllowanceResetUpgradeable {
```
When `AccessControlUpgradeable` has such gap:
```solidity
node_modules/@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol
259:     uint256[49] private __gap;
```

```solidity
contracts/utils/SafeAllowanceResetUpgradeable.sol
9: abstract contract SafeAllowanceResetUpgradeable {
```
 does not.

## Tool used

Manual Review

## Recommendation
Add `uint256[50] private __gap;` to base contracts, which are upgradable/inherted by upgradable contracts.
