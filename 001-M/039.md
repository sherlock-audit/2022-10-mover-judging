berndartmueller

medium

# Protocol does not work with fee-on-transfer tokens

## Summary

Fee-on-transfer tokens are not supported by the protocol. This leads to underfunded token transfers and the transactions to revert when swapping/bridging, preventing top-ups with fee-on transfer tokens.

## Vulnerability Detail

ERC20 tokens may make specific customizations to their ERC20 contracts.
One type of these tokens is deflationary tokens that charge a specific fee for every `transfer()` or `transferFrom()`.

In the current protocol implementation, both the `ExchangeProxy` and the `HardenedTopupProxy` contracts assume that the received amount is the same as the transfer amount, and use it for internal balance bookkeeping.

However, if the token is a fee-on transfer token, the actually received amount can differ and will be less than the initial transfer amount. This will cause the transaction to revert due to insufficient tokens available for bridging/swapping.

## Impact

The transaction will revert when trying to bridge or swap tokens that are fee-on transfer tokens, preventing the use of fee-on transfer tokens for top-ups.

## Code Snippet

[ExchangeProxy.sol#L99](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L99)

```solidity
function executeSwap(
  address _tokenFrom,
  address _tokenTo,
  uint256 _amount,
  bytes memory _data
) public payable override returns (uint256) {
  // native token doesn't need to be transferred explicitly, it's in tx.value
  if (_tokenFrom != ETH_TOKEN_ADDRESS) {
      IERC20(_tokenFrom).safeTransferFrom(msg.sender, address(this), _amount);
  }
  // after token is transferred to this contract, call actual swap
  return executeSwapDirect(msg.sender, _tokenFrom, _tokenTo, _amount, 0, _data);
}
```

[HardenedTopupProxy.sol#L315-L343](https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/HardenedTopupProxy.sol#L315-L343)

```solidity
function _processTopup(address _beneficiary, address _token, uint256 _amount, uint256 _expectedMinimumReceived, bytes memory _convertData, uint256 _bridgeType, bytes memory _bridgeTxData, bytes32 _receiverHash) internal
{
  // don't go further is contract function is paused (by admin or pauser)
  require(paused == false, "operations paused");

  // if execution passes to here, it means:
  // 1. operations are active;
  // 2. allowance check is passed, allowance is set and we can use funds from _beneficiary's address;
  // next steps are:
  // 1. transfer token to this contract or exchange proxy
  //    TODO (future version): check if token must be unwrapped by a partner
  //    (vault tokens, etc. that could not be swapped and should be unwrapped in some way)
  // 2. swap if needed and transfer USDC to this contract
  // 3. deduct topup fee (if needed)
  // 4. check allowance to the bridge contract
  // 5. check the bridge address is in the whitelist and perform a call to bridge

  // if the token to be provided matches the defined cardTopupToken, do not perform any
  // swap, just deduct topup fee (if fee > 0) and proceed to bridging for settlement
  if (_token == cardTopupToken) {
      // beneficiary is msg.sender (perform static check)
      IERC20Upgradeable(_token).safeTransferFrom(_beneficiary, address(this), _amount); // @audit-info `_amount` is transferred

      uint256 feeAmount = _amount.mul(topupFee).div(1e18); // @audit-info `_amount` received could be less than anticipated for fee-on transfer tokens

      // bridge from _beneficiary to card L1 relay
      _bridgeAssetDirect(_amount.sub(feeAmount), _bridgeType, _bridgeTxData);

      emit CardTopup(_beneficiary, _token, _amount, _amount.sub(feeAmount), _receiverHash);
      return;
  }

  // conversion is required, perform swap through exchangeProxy
  if (_token != ETH_TOKEN_ADDRESS) {
      // transfer tokens on the exchange proxy balance before performing swap call
      IERC20Upgradeable(_token).safeTransferFrom(_beneficiary, address(exchangeProxyContract), _amount); // @audit-info `_amount` is transferred
  }

  // exchange proxy is trusted and would check swap provider on its own in trusted registry contract
  uint256 amountReceived =
      IExchangeProxy(address(exchangeProxyContract)).executeSwapDirect{value: msg.value}(
          address(this),
          _token,
          cardTopupToken,
          _amount, // @audit-info `_amount` received could be less than anticipated for fee-on transfer tokens
          exchangeFee,
          _convertData
      );

  [..]
}
```

## Tool Used

Manual Review

## Recommendation

Consider using the actually received amount by calculating the difference in token balance before and after the token transfer.
