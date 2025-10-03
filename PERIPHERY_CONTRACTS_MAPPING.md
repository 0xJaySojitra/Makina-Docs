# Periphery Contracts Public Functions and Modifiers Mapping

This document provides a comprehensive mapping of all periphery contracts with their public functions, modifiers, and descriptions for spreadsheet creation.

## Contract Overview

The Makina periphery contracts provide additional functionality and services that extend the core protocol, including security modules, depositors, redeemers, fee managers, and utility contracts.

---

## 1. SecurityModule Contract

**File:** `makina-periphery/src/security-module/SecurityModule.sol`  
**Description:** ERC20 vault for staking machine shares to provide protocol security with slashing capabilities

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `bytes` | - | Initializes the security module |
| `decimals` | public | `pure` | - | `uint8` | Returns the token decimals |
| `machine` | public | `view` | - | `address` | Returns the machine address |
| `machineShare` | public | `view` | - | `address` | Returns the machine share token address |
| `cooldownReceipt` | public | `view` | - | `address` | Returns the cooldown receipt NFT address |
| `cooldownDuration` | public | `view` | - | `uint256` | Returns the cooldown duration |
| `maxSlashableBps` | public | `view` | - | `uint256` | Returns max slashable amount in basis points |
| `minBalanceAfterSlash` | public | `view` | - | `uint256` | Returns minimum balance after slash |
| `pendingCooldown` | external | `view` | `uint256` | `uint256, uint256, uint256` | Returns pending cooldown details |
| `slashingMode` | public | `view` | - | `bool` | Returns if slashing mode is active |
| `totalLockedAmount` | public | `view` | - | `uint256` | Returns total locked machine shares |
| `maxSlashable` | public | `view` | - | `uint256` | Returns maximum slashable amount |
| `convertToShares` | public | `view` | `uint256` | `uint256` | Converts assets to security module shares |
| `convertToAssets` | public | `view` | `uint256` | `uint256` | Converts security module shares to assets |
| `previewLock` | public | `view, NotSlashingMode` | `uint256` | `uint256` | Previews shares for locking assets |
| `lock` | external | `nonReentrant, NotSlashingMode` | `uint256, address, uint256` | `uint256` | Locks machine shares and mints security shares |
| `startCooldown` | external | `nonReentrant` | `uint256, address` | `uint256, uint256, uint256` | Starts cooldown for redemption |
| `cancelCooldown` | external | `nonReentrant` | `uint256` | `uint256` | Cancels an active cooldown |
| `redeem` | external | `nonReentrant` | `uint256, uint256` | `uint256` | Redeems after cooldown completion |
| `slash` | external | `nonReentrant, onlySecurityCouncil` | `uint256` | - | Slashes locked machine shares |
| `settleSlashing` | external | `onlySecurityCouncil` | - | - | Settles slashing and exits slashing mode |
| `setCooldownDuration` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the cooldown duration |
| `setMaxSlashableBps` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets max slashable basis points |
| `setMinBalanceAfterSlash` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets minimum balance after slash |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `NotSlashingMode` | Only allows execution when not in slashing mode |
| `nonReentrant` | Prevents reentrancy attacks |
| `onlySecurityCouncil` | Only allows security council to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `initializer` | Ensures function can only be called once during initialization |

---

## 2. DirectDepositor Contract

**File:** `makina-periphery/src/depositors/DirectDepositor.sol`  
**Description:** Simple depositor that directly deposits assets into the machine with optional whitelist

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `bytes` | - | Initializes the depositor |
| `deposit` | public | `whitelistCheck` | `uint256, address, uint256` | `uint256` | Deposits assets into the machine |
| `setWhitelistStatus` | external | `onlyRiskManager` | `bool` | - | Enables/disables whitelist |
| `setWhitelistedUsers` | external | `onlyRiskManager` | `address[], bool` | - | Sets whitelist status for users |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `whitelistCheck` | Checks whitelist requirements if enabled |
| `onlyRiskManager` | Only allows risk manager to execute |
| `initializer` | Ensures function can only be called once during initialization |

---

## 3. AsyncRedeemer Contract

**File:** `makina-periphery/src/redeemers/AsyncRedeemer.sol`  
**Description:** ERC721-based async redemption system with finalization delays and batch processing

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `bytes` | - | Initializes the async redeemer |
| `nextRequestId` | external | `view` | - | `uint256` | Returns the next request ID |
| `lastFinalizedRequestId` | external | `view` | - | `uint256` | Returns the last finalized request ID |
| `finalizationDelay` | external | `view` | - | `uint256` | Returns the finalization delay |
| `getShares` | external | `view` | `uint256` | `uint256` | Returns shares for a request ID |
| `getClaimableAssets` | public | `view` | `uint256` | `uint256` | Returns claimable assets for a request |
| `previewFinalizeRequests` | public | `view` | `uint256` | `uint256, uint256` | Previews finalization of requests |
| `requestRedeem` | public | `nonReentrant, whitelistCheck` | `uint256, address` | `uint256` | Creates a redemption request |
| `finalizeRequests` | external | `onlyMechanic, nonReentrant` | `uint256, uint256` | `uint256, uint256` | Finalizes redemption requests |
| `claimAssets` | external | `nonReentrant, whitelistCheck` | `uint256` | `uint256` | Claims finalized assets |
| `setFinalizationDelay` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the finalization delay |
| `setWhitelistStatus` | external | `onlyRiskManager` | `bool` | - | Enables/disables whitelist |
| `setWhitelistedUsers` | external | `onlyRiskManager` | `address[], bool` | - | Sets whitelist status for users |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `nonReentrant` | Prevents reentrancy attacks |
| `whitelistCheck` | Checks whitelist requirements if enabled |
| `onlyMechanic` | Only allows mechanic role to execute |
| `onlyRiskManager` | Only allows risk manager to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `initializer` | Ensures function can only be called once during initialization |

---

## 4. WatermarkFeeManager Contract

**File:** `makina-periphery/src/fee-managers/WatermarkFeeManager.sol`  
**Description:** Fee manager implementing watermark-based performance fees and management fees

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `bytes` | - | Initializes the fee manager |
| `authority` | public | `view` | - | `address` | Returns the access manager authority |
| `mgmtFeeRatePerSecond` | external | `view` | - | `uint256` | Returns management fee rate per second |
| `smFeeRatePerSecond` | external | `view` | - | `uint256` | Returns security module fee rate per second |
| `perfFeeRate` | external | `view` | - | `uint256` | Returns performance fee rate |
| `mgmtFeeReceivers` | external | `view` | - | `address[]` | Returns management fee receivers |
| `mgmtFeeSplitBps` | external | `view` | - | `uint256[]` | Returns management fee split in basis points |
| `perfFeeReceivers` | external | `view` | - | `address[]` | Returns performance fee receivers |
| `perfFeeSplitBps` | external | `view` | - | `uint256[]` | Returns performance fee split in basis points |
| `securityModule` | external | `view` | - | `address` | Returns the security module address |
| `sharePriceWatermark` | external | `view` | - | `uint256` | Returns the share price watermark |
| `calculateFixedFee` | external | `view` | `uint256, uint256` | `uint256` | Calculates fixed management fees |
| `calculatePerformanceFee` | external | `onlyMachine` | `uint256, uint256, uint256, uint256` | `uint256` | Calculates performance fees |
| `distributeFees` | external | `onlyMachine` | `uint256, uint256` | - | Distributes fees to receivers |
| `resetSharePriceWatermark` | external | `restricted` | `uint256` | - | Resets the share price watermark |
| `setMgmtFeeRatePerSecond` | external | `restricted` | `uint256` | - | Sets management fee rate per second |
| `setSmFeeRatePerSecond` | external | `restricted` | `uint256` | - | Sets security module fee rate per second |
| `setPerfFeeRate` | external | `restricted` | `uint256` | - | Sets performance fee rate |
| `setMgmtFeeSplit` | external | `restricted` | `address[], uint256[]` | - | Sets management fee split |
| `setPerfFeeSplit` | external | `restricted` | `address[], uint256[]` | - | Sets performance fee split |
| `setSecurityModule` | external | `onlyFactory` | `address` | - | Sets the security module address |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyMachine` | Only allows the machine to execute |
| `onlyFactory` | Only allows factory to execute |
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 5. HubPeripheryFactory Contract

**File:** `makina-periphery/src/factories/HubPeripheryFactory.sol`  
**Description:** Factory for creating periphery contracts like depositors, redeemers, fee managers, and security modules

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `address` | - | Initializes the factory |
| `isDepositor` | external | `view` | `address` | `bool` | Checks if address is a depositor |
| `isRedeemer` | external | `view` | `address` | `bool` | Checks if address is a redeemer |
| `isFeeManager` | external | `view` | `address` | `bool` | Checks if address is a fee manager |
| `isSecurityModule` | external | `view` | `address` | `bool` | Checks if address is a security module |
| `depositorImplemId` | external | `view` | `address` | `uint16` | Returns depositor implementation ID |
| `redeemerImplemId` | external | `view` | `address` | `uint16` | Returns redeemer implementation ID |
| `feeManagerImplemId` | external | `view` | `address` | `uint16` | Returns fee manager implementation ID |
| `setMachine` | external | `restricted` | `address, address` | - | Sets machine for periphery contract |
| `setSecurityModule` | external | `restricted` | `address, address` | - | Sets security module for fee manager |
| `createDepositor` | external | `restricted` | `uint16, bytes` | `address` | Creates a depositor contract |
| `createRedeemer` | external | `restricted` | `uint16, bytes` | `address` | Creates a redeemer contract |
| `createFeeManager` | external | `restricted` | `uint16, bytes` | `address` | Creates a fee manager contract |
| `createSecurityModule` | external | `restricted` | `SecurityModuleInitParams` | `address` | Creates a security module contract |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 6. FlashloanAggregator Contract

**File:** `makina-periphery/src/flashloans/FlashloanAggregator.sol`  
**Description:** Aggregates multiple flashloan providers (Balancer V2/V3, Morpho, Maker DSS Flash, Aave V3)

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `requestFlashloan` | external | `onlyCaliber` | `FlashloanRequest` | - | Requests flashloan from specified provider |
| `receiveFlashLoan` | external | - | `BalancerIERC20[], uint256[], uint256[], bytes` | - | Balancer V2 flashloan callback |
| `balancerV3FlashloanCallback` | external | - | `address, ICaliber.Instruction, address, uint256` | - | Balancer V3 flashloan callback |
| `onMorphoFlashLoan` | external | - | `uint256, bytes` | - | Morpho flashloan callback |
| `onFlashLoan` | external | - | `address, address, uint256, uint256, bytes` | `bytes32` | ERC3156 flashloan callback (DSS Flash) |
| `executeOperation` | external | - | `address, uint256, uint256, address, bytes` | `bool` | Aave V3 flashloan callback |
| `ADDRESSES_PROVIDER` | external | `view` | - | `IPoolAddressesProvider` | Returns Aave addresses provider |
| `POOL` | external | `view` | - | `IPool` | Returns Aave pool |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyCaliber` | Only allows caliber contracts to execute |

---

## 7. SMCooldownReceipt Contract

**File:** `makina-periphery/src/security-module/SMCooldownReceipt.sol`  
**Description:** ERC721 NFT representing cooldown periods in the security module

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `mint` | external | `onlyOwner` | `address` | `uint256` | Mints a cooldown receipt NFT |
| `burn` | external | `onlyOwner` | `uint256` | - | Burns a cooldown receipt NFT |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyOwner` | Only allows the owner to execute |

---

## 8. ERC4626Oracle Contract

**File:** `makina-periphery/src/oracles/ERC4626Oracle.sol`  
**Description:** Oracle for ERC4626 vault tokens that provides price feeds based on underlying assets

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `bytes` | - | Initializes the oracle |
| `vault` | external | `view` | - | `address` | Returns the ERC4626 vault address |
| `underlyingOracle` | external | `view` | - | `address` | Returns the underlying asset oracle |
| `getPrice` | external | `view` | `address, address` | `uint256` | Returns price for token pair |
| `setVault` | external | `restricted` | `address` | - | Sets the ERC4626 vault address |
| `setUnderlyingOracle` | external | `restricted` | `address` | - | Sets the underlying oracle address |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 9. Whitelist Utility Contract

**File:** `makina-periphery/src/utils/Whitelist.sol`  
**Description:** Utility contract providing whitelist functionality for access control

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `whitelistEnabled` | external | `view` | - | `bool` | Returns if whitelist is enabled |
| `isWhitelisted` | external | `view` | `address` | `bool` | Checks if address is whitelisted |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `whitelistCheck` | Checks whitelist requirements if enabled |

---

## 10. MachinePeriphery Utility Contract

**File:** `makina-periphery/src/utils/MachinePeriphery.sol`  
**Description:** Base contract for periphery contracts that interact with machines

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `machine` | public | `view` | - | `address` | Returns the machine address |
| `setMachine` | external | `onlyFactory` | `address` | - | Sets the machine address |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyFactory` | Only allows factory to execute |
| `onlyMechanic` | Only allows mechanic role to execute |
| `onlyRiskManager` | Only allows risk manager to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `onlySecurityCouncil` | Only allows security council to execute |

---

## Summary

The periphery contracts provide extended functionality for:

1. **Security**: SecurityModule enables staking with slashing for protocol security
2. **User Access**: Depositors and redeemers provide user-friendly interfaces with optional whitelisting
3. **Fee Management**: WatermarkFeeManager implements sophisticated fee structures
4. **Flash Loans**: FlashloanAggregator provides access to multiple flashloan providers
5. **Oracles**: ERC4626Oracle enables price feeds for vault tokens
6. **Factory Pattern**: HubPeripheryFactory creates and manages periphery instances
7. **Utilities**: Base contracts provide common functionality like whitelisting and machine integration

All periphery contracts integrate with the core protocol while maintaining modular design and comprehensive access controls.

