ballx

high

# INCORRECT ACCESS CONTROL

## Summary
Access control plays an important role in segregation of privileges in smart contracts and other applications. If this is misconfigured or not properly validated on sensitive functions, it may lead to loss of funds, tokens and in some cases compromise of the smart contract.

The contract ExchangeProxy is importing an access control library ```@openzeppelin/contracts/access/AccessControl.sol but the function claimFees is missing the modifier onlyRole.```



## Vulnerability Detail

## Impact

## Code Snippet

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L242-L249

```solidity
    function claimFees(address _token, uint256 _amount) public {
        require(msg.sender == yieldDistributorAddress, "yield distributor only");
        if (_token != ETH_TOKEN_ADDRESS) {
            IERC20(_token).safeTransfer(msg.sender, _amount);
        } else {
            payable(msg.sender).sendValue(_amount);
        }
    }
```

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L131-L210
```solidity
    function executeSwapDirect(
        address _beneficiary,
        address _tokenFrom,
        address _tokenTo,
        uint256 _amount,
        uint256 _exchangeFee,
        bytes memory _data
    ) public payable override returns (uint256) {
        require(msg.sender == transferProxyAddress, "transfer proxy only");

        // extract values from bytes array provided
        address executorAddress;
        address spenderAddress;
        uint256 ethValue;

        bytes memory callData = ByteUtil.slice(_data, 72, _data.length - 72);
        assembly {
            executorAddress := mload(add(_data, add(0x14, 0)))
            spenderAddress := mload(add(_data, add(0x14, 0x14)))
            ethValue := mload(add(_data, add(0x20, 0x28)))
        }

        // allow spender to transfer tokens from this contract
        if (_tokenFrom != ETH_TOKEN_ADDRESS && spenderAddress != address(0)) {
            require(trustedRegistryContract.isWhitelisted(spenderAddress), "allowance to non-trusted");
            resetAllowanceIfNeeded(IERC20(_tokenFrom), spenderAddress, _amount);
        }

        // remember the actual balance of target token (fees could reside on this contract balance)
        uint256 balanceBefore = 0;
        if (_tokenTo != ETH_TOKEN_ADDRESS) {
            balanceBefore = IERC20(_tokenTo).balanceOf(address(this));
        } else {
            balanceBefore = address(this).balance;
        }

        // regardless of stated amount, the ETH value passed to exchange call must be provided to the contract
        require(msg.value >= ethValue, "insufficient ETH provided");

        // don't allow to call non-trusted addresses
        require(trustedRegistryContract.isWhitelisted(executorAddress), "call to non-trusted");

        // ensure no state passed, no reentrancy, etc.
        (bool success, ) = executorAddress.call{value: ethValue}(callData);
        require(success, "SWAP_CALL_FAILED");

        // always rely only on actual amount received regardless of called parameters
        uint256 amountReceived = 0;
        if (_tokenTo != ETH_TOKEN_ADDRESS) {
            amountReceived = IERC20(_tokenTo).balanceOf(address(this));
        } else {
            amountReceived = address(this).balance;
        }
        amountReceived = amountReceived.sub(balanceBefore);

        require(amountReceived > 0, "zero amount received");

        // process exchange fee if present (in deposit we get pool tokens, so process fees after swap, here we take fees in source token)
        // fees are left on this contract address and are harvested by yield distributor
        //uint256 feeAmount = amountReceived.mul(_exchangeFee).div(1e18);
        amountReceived = amountReceived.sub(
            amountReceived.mul(_exchangeFee).div(1e18)
        ); // this is return value that should reflect actual result of swap (for deposit, etc.)

        if (_tokenTo != ETH_TOKEN_ADDRESS) {
            //send received tokens to beneficiary directly
            IERC20(_tokenTo).safeTransfer(_beneficiary, amountReceived);
        } else {
            //send received eth to beneficiary directly
            payable(_beneficiary).sendValue(amountReceived);
            // payable(_beneficiary).transfer(amountReceived);
            // should work for external wallets (currently is the case)
            // but wont work for some other smart contracts due to gas stipend limit
        }

        emit ExecuteSwap(_beneficiary, _tokenFrom, _tokenTo, _amount, amountReceived);
        
        // amount received is used to check minimal amount condition set by calling app from topup proxy contract
        return amountReceived;
    }

```

https://github.com/sherlock-audit/2022-10-mover/blob/main/cardtopup_contract/contracts/ExchangeProxy.sol#L91-L103

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
## Tool used

Manual Review

## Recommendation

It is recommended to go through the contract and observe the functions that are lacking an access control modifier. If they contain sensitive administrative actions, it is advised to add a suitable modifier to the same
