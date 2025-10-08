# Complete User Flow Guide for Makina Protocol

## Overview

This document provides a comprehensive explanation of user interactions with the Makina protocol at the contract level, including complete flow examples for common operations like depositing, strategy execution, cross-chain transfers, and redemption.

## Architecture Context

The Makina protocol operates as a cross-chain vault system with the following key components:
- **Machine**: Central vault managing deposits, redemptions, and cross-chain operations
- **Caliber**: Strategy execution engine using Weiroll VM for complex DeFi operations
- **Security Module**: Staking mechanism for protocol security with slashing capabilities
- **Bridge Adapters**: Cross-chain communication and asset transfers

## 1. User Deposit Flow (Direct Deposit)

### Contract-Level Flow

```
User ────approve()───► ERC20 Token (USDC/USDT/DAI)
    │
    └───deposit()───► DirectDepositor ───deposit()───► Machine
```

### Complete Example with Contract Calls

```solidity
// 1. User approves token spending
IERC20(assetToken).approve(directDepositorAddress, depositAmount);

// 2. User initiates deposit through DirectDepositor
IDirectDepositor(depositorAddress).deposit(
    depositAmount,    // uint256 assets
    receiverAddress,  // address receiver
    minSharesOut      // uint256 minSharesOut
);

// 3. DirectDepositor calls Machine.deposit()
IMachine(machineAddress).deposit(
    depositAmount,    // uint256 assets
    receiverAddress,  // address receiver
    minSharesOut      // uint256 minSharesOut
);

// 4. Machine calculates shares using current AUM
uint256 shares = convertToShares(depositAmount);

// 5. Machine transfers tokens from depositor
IERC20(assetToken).transferFrom(depositorAddress, machineAddress, depositAmount);

// 6. Machine mints shares to receiver
IMachineShare(shareToken).mint(receiverAddress, shares);

// 7. DirectDepositor returns shares to user
return shares;
```

### Key State Changes
- **Machine Total Assets**: Increases by `depositAmount`
- **Machine Total Supply**: Increases by `shares`
- **User Balance**: Receives `shares` of machine tokens

## 2. Strategy Execution Flow (Mechanic Operation)

### Contract-Level Flow

```
Mechanic ───managePosition()───► Caliber ───execute Weiroll VM───► External Protocols
```

### Complete Example with Strategy Execution

```solidity
// 1. Mechanic prepares management instruction
ICaliber.Instruction memory mgmtInstr = ICaliber.Instruction({
    program: managementProgram,  // bytes weiroll program
    inputs: managementInputs     // bytes[] program inputs
});

// 2. Mechanic prepares accounting instruction
ICaliber.Instruction memory acctInstr = ICaliber.Instruction({
    program: accountingProgram,  // bytes weiroll program
    inputs: accountingInputs     // bytes[] program inputs
});

// 3. Mechanic calls managePosition
ICaliber(caliberAddress).managePosition(
    mgmtInstr,  // Instruction management instruction
    acctInstr   // Instruction accounting instruction
);

// 4. Caliber verifies Merkle proof (if required)
bytes32[] memory proof = generateMerkleProof(instructions, allowedInstrRoot);
require(verifyMerkleProof(instructions, proof, allowedInstrRoot));

// 5. Caliber executes management instruction via Weiroll VM
IWeirollVM(weirollVm).execute(
    managementProgram,
    managementInputs,
    address(this)  // Caliber address as executor
);

// 6. Weiroll VM executes commands (example: Uniswap V3 swap)
IUniswapV3Pool(pool).swap(
    recipient,      // address recipient
    zeroForOne,     // bool zeroForOne
    amountSpecified, // int256 amountSpecified
    sqrtPriceLimitX96, // uint160 sqrtPriceLimitX96
    abi.encode(callbackData) // bytes data
);

// 7. Caliber executes accounting instruction
IWeirollVM(weirollVm).execute(
    accountingProgram,
    accountingInputs,
    address(this)
);

// 8. Accounting queries position value (example: Aave position)
IAavePool(aavePool).getUserAccountData(userAddress);

// 9. Caliber validates position changes and updates state
require(positionValueChange <= maxPositionIncreaseLossBps);
position.value = newValue;
position.lastAccountingTime = block.timestamp;
```

### Key State Changes
- **Position Values**: Updated based on strategy performance
- **Base Token Balances**: Modified through strategy operations
- **Machine AUM**: Recalculated after position updates

## 3. Cross-Chain Transfer Flow

### Contract-Level Flow (Hub to Spoke)

```
Mechanic ──transferToSpokeCaliber()──► Machine ──sendOutBridgeTransfer()──► BridgeAdapter ──► External Bridge
```

### Complete Example with Bridge Transfer

```solidity
// 1. Mechanic initiates spoke transfer through Machine
IMachine(machineAddress).transferToSpokeCaliber(
    spokeChainId,     // uint16 spokeChainId
    transferAmount,   // uint256 amount
    spokeAsset,       // address spokeAsset
    bridgeFee,        // uint256 bridgeFee
    bridgeId          // uint256 bridgeId
);

// 2. Machine validates transfer parameters
require(isOutTransferEnabled(bridgeId));
require(transferAmount <= maxBridgeLossBps);

// 3. Machine calls bridge adapter
IBridgeAdapter(bridgeAdapter).sendOutBridgeTransfer(
    spokeChainId,     // uint16 chainId
    transferAmount,   // uint256 amount
    bridgeData        // bytes bridge-specific data
);

// 4. Bridge adapter interacts with external bridge (example: Across)
IAcrossSpokePool(spokePool).deposit(
    recipient,        // address recipient
    token,            // address token
    amount,           // uint256 amount
    depositId         // uint256 depositId
);

// 5. On destination chain, CaliberMailbox handles incoming transfer
ICaliberMailbox(mailboxAddress).manageTransfer(
    tokenAddress,     // address token
    transferAmount,   // uint256 amount
    bridgeData        // bytes bridge data
);

// 6. CaliberMailbox authorizes and claims transfer
ICaliberMailbox(mailboxAddress).authorizeInBridgeTransfer(
    bridgeId,         // uint16 bridgeId
    transferId        // bytes32 transferId
);

// 7. CaliberMailbox claims tokens for Caliber
ICaliberMailbox(mailboxAddress).claimInBridgeTransfer(
    bridgeId,         // uint16 bridgeId
    transferAmount    // uint256 amount
);

// 8. Caliber receives tokens and updates position
ICaliber(caliberAddress).notifyIncomingTransfer(
    tokenAddress,     // address token
    transferAmount    // uint256 amount
);
```

### Key State Changes
- **Machine Assets**: Temporarily locked for bridge transfer
- **Spoke Caliber**: Receives assets for strategy execution
- **Bridge State**: Updates transfer tracking and status

## 4. Security Module Participation Flow

### Contract-Level Flow (Staking Process)

```
User ────approve()───► MachineShare ────lock()───► SecurityModule ────mint()───► SecurityShare
```

### Complete Example with Security Module

```solidity
// 1. User approves machine share spending
IMachineShare(machineShare).approve(securityModuleAddress, lockAmount);

// 2. User locks shares in Security Module
ISecurityModule(securityModule).lock(
    lockAmount,       // uint256 shares
    receiverAddress,  // address receiver
    minSharesOut      // uint256 minSharesOut
);

// 3. Security Module transfers shares from user
IMachineShare(machineShare).transferFrom(userAddress, securityModuleAddress, lockAmount);

// 4. Security Module mints security shares to user
ISecurityModule(securityModule).mint(receiverAddress, securityShares);

// 5. Security Module updates total locked amount
totalLockedAmount += lockAmount;

// 6. User starts cooldown for redemption (after holding period)
ISecurityModule(securityModule).startCooldown(
    redeemShares,     // uint256 shares
    receiverAddress   // address receiver
);

// 7. Security Module burns security shares and mints cooldown NFT
ISecurityModule(securityModule).burn(userAddress, redeemShares);
ISMCooldownReceipt(cooldownReceipt).mint(receiverAddress, cooldownId);

// 8. After cooldown period, user redeems
ISecurityModule(securityModule).redeem(
    cooldownId,       // uint256 cooldownId
    minAssetsOut      // uint256 minAssetsOut
);

// 9. Security Module validates cooldown maturity
require(block.timestamp >= cooldownMaturity);

// 10. Security Module calculates redeemable assets
uint256 assetsOut = convertToAssets(redeemShares);

// 11. Security Module transfers machine shares back to user
IMachineShare(machineShare).transfer(receiverAddress, redeemShares);

// 12. Security Module burns cooldown NFT
ISMCooldownReceipt(cooldownReceipt).burn(cooldownId);
```

### Key State Changes
- **Security Module**: Locks machine shares, mints security shares
- **Machine**: Total supply and AUM remain unchanged during staking
- **User**: Exchanges machine shares for security shares (and vice versa)

## 5. Fee Collection and Distribution Flow

### Contract-Level Flow

```
Machine ────calculateFees()───► WatermarkFeeManager ────distributeFees()───► Fee Receivers
```

### Complete Example with Fee Management

```solidity
// 1. Machine calculates performance fees on AUM increase
IWatermarkFeeManager(feeManager).calculatePerformanceFee(
    currentAUM,       // uint256 currentAUM
    lastAUM,          // uint256 lastAUM
    sharePrice,       // uint256 sharePrice
    totalShares       // uint256 totalShares
);

// 2. Fee Manager checks watermark for performance fees
if (sharePrice > sharePriceWatermark) {
    uint256 profit = sharePrice - sharePriceWatermark;
    uint256 feeShares = (profit * totalShares * perfFeeRate) / 1e18;

    // Mint fee shares to fee receivers
    for (uint256 i = 0; i < perfFeeReceivers.length; i++) {
        uint256 receiverShares = (feeShares * perfFeeSplitBps[i]) / 10000;
        IMachineShare(shareToken).mint(perfFeeReceivers[i], receiverShares);
    }
}

// 3. Fee Manager calculates management fees
uint256 mgmtFeeAssets = calculateFixedFee(
    timeElapsed,      // uint256 timeElapsed
    mgmtFeeRatePerSecond // uint256 mgmtFeeRatePerSecond
);

// 4. Machine distributes management fees
IMachine(machineAddress).distributeFees(
    mgmtFeeAssets,    // uint256 mgmtFeeAssets
    perfFeeAssets     // uint256 perfFeeAssets
);

// 5. Fee Manager transfers assets to receivers
for (uint256 i = 0; i < mgmtFeeReceivers.length; i++) {
    uint256 receiverAssets = (mgmtFeeAssets * mgmtFeeSplitBps[i]) / 10000;
    IERC20(assetToken).transfer(mgmtFeeReceivers[i], receiverAssets);
}
```

### Key State Changes
- **Fee Receivers**: Receive newly minted shares or asset transfers
- **Machine Total Supply**: Increases with performance fee minting
- **Share Price Watermark**: Updated for future fee calculations

## 6. Emergency Procedures Flow

### Contract-Level Flow (Recovery Mode)

```
SecurityCouncil ────pause()───► Machine/Caliber ────Emergency Stop───► All Operations
```

### Complete Example with Emergency Procedures

```solidity
// 1. Security Council triggers recovery mode
IMakinaGovernable(machineAddress).setRecoveryMode(true);

// 2. All critical operations are paused
// - Deposits and redemptions blocked
// - Strategy execution halted
// - Bridge transfers disabled

// 3. Risk Manager updates critical parameters during recovery
IRiskManager(riskManager).updateEmergencyParameters(
    newMaxBridgeLossBps,    // uint256 maxBridgeLossBps
    newPositionLimits,      // uint256[] positionLimits
    newFeeParameters        // bytes feeParameters
);

// 4. Security Council resets bridge state if needed
IMachine(machineAddress).resetBridgingState(problematicToken);

// 5. Security Council settles slashing if protocol was exploited
ISecurityModule(securityModule).settleSlashing();

// 6. Once resolved, Security Council exits recovery mode
IMakinaGovernable(machineAddress).setRecoveryMode(false);

// 7. Normal operations resume with updated parameters
```

### Key State Changes
- **Protocol State**: Toggles between normal and recovery modes
- **Bridge State**: Can be reset for specific tokens
- **Security Module**: Slashing can be settled and exited
- **Risk Parameters**: Can be updated during emergency

## 7. Cross-Chain Accounting Update Flow

### Contract-Level Flow

```
Spoke Caliber ────updateAccounting()───► Machine ────Sync AUM───► Hub Accounting
```

### Complete Example with Accounting Updates

```solidity
// 1. Spoke Caliber prepares accounting data
SpokeCaliberAccountingData memory accountingData = SpokeCaliberAccountingData({
    totalBaseTokenValue: calculateTotalValue(),  // uint256 totalBaseTokenValue
    positionDetails: getPositionDetails(),       // bytes[] positionDetails
    baseTokenDetails: getBaseTokenDetails(),     // bytes[] baseTokenDetails
    timestamp: block.timestamp                   // uint256 timestamp
});

// 2. Spoke Caliber calls Machine update
IMachine(machineAddress).updateSpokeCaliberAccountingData(
    encodedAccountingData,  // bytes encoded data
    guardianSignatures      // GuardianSignature[] signatures
);

// 3. Machine validates guardian signatures
require(validateGuardianSignatures(encodedData, signatures, threshold));

// 4. Machine decodes and validates accounting data
require(accountingData.timestamp > lastGlobalAccountingTime);
require(accountingData.totalBaseTokenValue <= maxExpectedValue);

// 5. Machine updates spoke caliber data
spokeCalibers[chainId] = accountingData;

// 6. Machine recalculates total AUM across all chains
uint256 newTotalAUM = hubBaseTokenValue + spokeBaseTokenValues;
lastTotalAUM = newTotalAUM;
lastGlobalAccountingTime = block.timestamp;

// 7. Machine emits accounting update event
emit SpokeCaliberAccountingUpdated(chainId, newTotalAUM);
```

### Key State Changes
- **Global AUM**: Updated with latest spoke chain values
- **Accounting Timestamp**: Updated for staleness checks
- **Spoke Data**: Refreshed with latest position values

## Integration Points and Dependencies

### External Protocol Interactions
1. **DEX Protocols**: Uniswap V3, Sushiswap for swaps in Caliber strategies
2. **Lending Protocols**: Aave, Compound for position management
3. **Bridge Protocols**: Across, Wormhole, Hyperlane for cross-chain transfers
4. **Oracle Networks**: Chainlink, custom oracles for price feeds
5. **Flash Loan Providers**: Balancer, Morpho, Maker, Aave for leverage

### Access Control Integration
1. **Role Validation**: All operations validate caller permissions
2. **Timelock Delays**: Sensitive operations require time delays
3. **Governance Coordination**: Multi-role governance for critical changes
4. **Emergency Procedures**: Security council override capabilities

This comprehensive flow documentation provides auditors and developers with complete visibility into how users interact with the Makina protocol at the contract level, enabling thorough security analysis and testing.
