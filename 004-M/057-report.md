caventa

medium

# Nonces not used in signed data

## Summary
Nonces are not used in the signature checks

## Vulnerability Detail
A nonce can prevent an old value from being used when a new value exists. Without one, two transactions submitted in one order can appear in a block in a different order

## Impact
If a user is attacked, then tries to change the recipient address to a more secure address, initially chooses an insecure compromised one, but immediately notices the problem, then re-submits as a different, uncompromised address, a malicious miner can change the order of the transactions, so the insecure one is the one that ends up taking effect, letting the attacker transfer the funds

## Code Snippet
https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L508-L514

## Tool used
Manual Review

## Recommendation
Include a nonce in what is signed