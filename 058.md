pashov

high

# `safeIncreaseAllowance` will result in a DoS with USDT (and other such non-ERC20 conforming tokens)

## Summary
In both `SafeAllowanceReset::resetAllowanceIfNeeded` and `SafeAllowanceResetUpgradeable::resetAllowanceIfNeeded` the code makes use of the `safeIncreaseAllowance` functionality. The problem is with [USDT](https://gist.github.com/plutoegg/a8794a24dfa84d0b0104141612b52977#file-tethertoken-sol-L265) as you can see in its `approve` function. USDT expects each call to `approve` to first zero out the allowance and only then you can change it.

## Vulnerability Detail
The code does not zero out the allowance first so it will always revert when using USDT.

## Impact
This vulnerability results in a DoS in using the basic functionalities of the protocol and it will happen often because USDT is an oftenly used token.

## Code Snippet
https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/utils/SafeAllowanceReset.sol#L24
https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/utils/SafeAllowanceResetUpgradeable.sol#L24

## Tool used

Manual Review

## Recommendation
Always call `token.approve(spender, 0)` before approving any amount to be spent - this will allow USDT to not revert.
