# Core Contracts Public Functions and Modifiers Mapping

This document provides a comprehensive mapping of all core contracts with their public functions, modifiers, and descriptions for spreadsheet creation.

## Contract Overview

The Makina core contracts handle the fundamental protocol operations including machine management, caliber execution, bridge operations, and registry management.

---

## 1. Machine Contract

**File:** `makina-core/src/machine/Machine.sol`  
**Description:** Main vault contract that manages deposits, redemptions, and cross-chain operations

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `MachineInitParams, MakinaGovernableInitParams, address, address, address, address` | - | Initializes the machine with parameters |
| `depositor` | external | `view` | - | `address` | Returns the current depositor address |
| `redeemer` | external | `view` | - | `address` | Returns the current redeemer address |
| `shareToken` | external | `view` | - | `address` | Returns the share token address |
| `accountingToken` | external | `view` | - | `address` | Returns the accounting token address |
| `hubCaliber` | external | `view` | - | `address` | Returns the hub caliber address |
| `feeManager` | external | `view` | - | `address` | Returns the fee manager address |
| `caliberStaleThreshold` | external | `view` | - | `uint256` | Returns the caliber stale threshold |
| `maxFixedFeeAccrualRate` | external | `view` | - | `uint256` | Returns the max fixed fee accrual rate |
| `maxPerfFeeAccrualRate` | external | `view` | - | `uint256` | Returns the max performance fee accrual rate |
| `feeMintCooldown` | external | `view` | - | `uint256` | Returns the fee mint cooldown period |
| `shareLimit` | external | `view` | - | `uint256` | Returns the share limit |
| `maxMint` | public | `view` | - | `uint256` | Returns the maximum mintable shares |
| `maxWithdraw` | public | `view` | - | `uint256` | Returns the maximum withdrawable assets |
| `lastTotalAum` | external | `view` | - | `uint256` | Returns the last total AUM |
| `lastGlobalAccountingTime` | external | `view` | - | `uint256` | Returns the last global accounting time |
| `getSpokeCalibersLength` | external | `view` | - | `uint256` | Returns the number of spoke calibers |
| `getSpokeChainId` | external | `view` | `uint256` | `uint256` | Returns the chain ID for a spoke caliber |
| `getSpokeCaliberDetailedAum` | external | `view` | `uint256` | `uint256, bytes[], bytes[], uint256` | Returns detailed AUM for a spoke caliber |
| `getSpokeCaliberMailbox` | external | `view` | `uint256` | `address` | Returns the mailbox address for a spoke caliber |
| `getSpokeBridgeAdapter` | external | `view` | `uint256, uint16` | `address` | Returns the bridge adapter for a spoke |
| `isIdleToken` | external | `view` | `address` | `bool` | Checks if a token is idle |
| `convertToShares` | public | `view` | `uint256` | `uint256` | Converts assets to shares |
| `convertToAssets` | public | `view` | `uint256` | `uint256` | Converts shares to assets |
| `manageTransfer` | external | `nonReentrant` | `address, uint256, bytes` | - | Manages incoming transfers |
| `transferToHubCaliber` | external | `notRecoveryMode, onlyMechanic` | `address, uint256` | - | Transfers tokens to hub caliber |
| `transferToSpokeCaliber` | external | `nonReentrant, notRecoveryMode, onlyMechanic` | `uint16, uint256, address, uint256, uint256` | - | Transfers tokens to spoke caliber |
| `sendOutBridgeTransfer` | external | `notRecoveryMode, onlyMechanic` | `uint16, uint256, bytes` | - | Sends outbound bridge transfer |
| `authorizeInBridgeTransfer` | external | `onlyOperator` | `uint16, bytes32` | - | Authorizes inbound bridge transfer |
| `claimInBridgeTransfer` | external | `onlyOperator` | `uint16, uint256` | - | Claims inbound bridge transfer |
| `cancelOutBridgeTransfer` | external | `onlyOperator` | `uint16, uint256` | - | Cancels outbound bridge transfer |
| `updateTotalAum` | external | `nonReentrant, notRecoveryMode` | - | `uint256` | Updates the total AUM |
| `deposit` | external | `nonReentrant, notRecoveryMode` | `uint256, address, uint256` | `uint256` | Deposits assets and mints shares |
| `redeem` | external | `nonReentrant, notRecoveryMode` | `uint256, address, uint256` | `uint256` | Redeems shares for assets |
| `updateSpokeCaliberAccountingData` | external | `nonReentrant` | `bytes, GuardianSignature[]` | - | Updates spoke caliber accounting data |
| `setSpokeCaliber` | external | `restricted` | `uint256, address, uint16[], address[]` | - | Sets spoke caliber configuration |
| `setSpokeBridgeAdapter` | external | `restricted` | `uint256, uint16, address` | - | Sets spoke bridge adapter |
| `setDepositor` | external | `restricted` | `address` | - | Sets the depositor address |
| `setRedeemer` | external | `restricted` | `address` | - | Sets the redeemer address |
| `setFeeManager` | external | `restricted` | `address` | - | Sets the fee manager address |
| `setCaliberStaleThreshold` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the caliber stale threshold |
| `setMaxFixedFeeAccrualRate` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the max fixed fee accrual rate |
| `setMaxPerfFeeAccrualRate` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the max performance fee accrual rate |
| `setFeeMintCooldown` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets the fee mint cooldown |
| `setShareLimit` | external | `onlyRiskManager` | `uint256` | - | Sets the share limit |
| `setOutTransferEnabled` | external | `onlyRiskManagerTimelock` | `uint16, bool` | - | Enables/disables outbound transfers |
| `setMaxBridgeLossBps` | external | `onlyRiskManagerTimelock` | `uint16, uint256` | - | Sets max bridge loss in basis points |
| `resetBridgingState` | external | `onlySecurityCouncil` | `address` | - | Resets bridging state for a token |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `nonReentrant` | Prevents reentrancy attacks |
| `notRecoveryMode` | Only allows execution when not in recovery mode |
| `onlyMechanic` | Only allows mechanic role to execute |
| `onlyOperator` | Only allows operator role to execute |
| `onlyRiskManager` | Only allows risk manager role to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `onlySecurityCouncil` | Only allows security council to execute |
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 2. MachineShare Contract

**File:** `makina-core/src/machine/MachineShare.sol`  
**Description:** ERC20 token representing shares in the machine vault

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `decimals` | public | `pure` | - | `uint8` | Returns the token decimals |
| `minter` | external | `view` | - | `address` | Returns the minter address (owner) |
| `mint` | external | `onlyOwner` | `address, uint256` | - | Mints tokens to an address |
| `burn` | external | - | `address, uint256` | - | Burns tokens from an address |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyOwner` | Only allows the owner to execute |

---

## 3. Caliber Contract

**File:** `makina-core/src/caliber/Caliber.sol`  
**Description:** Execution engine for managing positions and strategies using Weiroll VM

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `CaliberInitParams, address, address` | - | Initializes the caliber |
| `authority` | public | `view` | - | `address` | Returns the access manager authority |
| `hubMachineEndpoint` | external | `view` | - | `address` | Returns the hub machine endpoint |
| `accountingToken` | external | `view` | - | `address` | Returns the accounting token |
| `positionStaleThreshold` | external | `view` | - | `uint256` | Returns the position stale threshold |
| `allowedInstrRoot` | public | `view` | - | `bytes32` | Returns the allowed instruction root |
| `timelockDuration` | external | `view` | - | `uint256` | Returns the timelock duration |
| `pendingAllowedInstrRoot` | public | `view` | - | `bytes32` | Returns the pending instruction root |
| `pendingTimelockExpiry` | public | `view` | - | `uint256` | Returns the pending timelock expiry |
| `maxPositionIncreaseLossBps` | external | `view` | - | `uint256` | Returns max position increase loss in bps |
| `maxPositionDecreaseLossBps` | external | `view` | - | `uint256` | Returns max position decrease loss in bps |
| `maxSwapLossBps` | external | `view` | - | `uint256` | Returns max swap loss in bps |
| `cooldownDuration` | external | `view` | - | `uint256` | Returns the cooldown duration |
| `getPositionsLength` | external | `view` | - | `uint256` | Returns the number of positions |
| `getPositionId` | external | `view` | `uint256` | `uint256` | Returns position ID at index |
| `getPosition` | external | `view` | `uint256` | `Position` | Returns position data |
| `isBaseToken` | external | `view` | `address` | `bool` | Checks if token is a base token |
| `getBaseTokensLength` | external | `view` | - | `uint256` | Returns number of base tokens |
| `getBaseToken` | external | `view` | `uint256` | `address` | Returns base token at index |
| `isInstrRootGuardian` | external | `view` | `address` | `bool` | Checks if address is instruction root guardian |
| `isAccountingFresh` | external | `view` | - | `bool` | Checks if accounting is fresh |
| `getDetailedAum` | external | `view` | - | `uint256, bytes[], bytes[]` | Returns detailed AUM breakdown |
| `addBaseToken` | external | `onlyRiskManagerTimelock` | `address` | - | Adds a base token |
| `removeBaseToken` | external | `onlyRiskManagerTimelock` | `address` | - | Removes a base token |
| `accountForPosition` | external | `nonReentrant` | `Instruction` | `uint256, int256` | Accounts for a single position |
| `accountForPositionBatch` | external | `nonReentrant` | `Instruction[], uint256[]` | `uint256[], int256[]` | Accounts for multiple positions |
| `managePosition` | external | `nonReentrant, onlyOperator` | `Instruction, Instruction` | `uint256, int256` | Manages a position |
| `managePositionBatch` | external | `nonReentrant, onlyOperator` | `Instruction[], Instruction[]` | `uint256[], int256[]` | Manages multiple positions |
| `manageFlashLoan` | external | - | `Instruction, address, uint256` | - | Manages flash loan execution |
| `harvest` | external | `nonReentrant, onlyOperator` | `Instruction, ISwapModule.SwapOrder[]` | - | Harvests rewards and swaps |
| `swap` | external | `nonReentrant, onlyOperator` | `ISwapModule.SwapOrder` | - | Executes a token swap |
| `transferToHubMachine` | external | `onlyOperator` | `address, uint256, bytes` | - | Transfers tokens to hub machine |
| `notifyIncomingTransfer` | external | `nonReentrant` | `address, uint256` | - | Notifies of incoming transfer |
| `setPositionStaleThreshold` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets position stale threshold |
| `setTimelockDuration` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets timelock duration |
| `scheduleAllowedInstrRootUpdate` | external | `onlyRiskManager` | `bytes32` | - | Schedules instruction root update |
| `cancelAllowedInstrRootUpdate` | external | - | - | - | Cancels instruction root update |
| `setMaxPositionIncreaseLossBps` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets max position increase loss |
| `setMaxPositionDecreaseLossBps` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets max position decrease loss |
| `setMaxSwapLossBps` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets max swap loss |
| `setCooldownDuration` | external | `onlyRiskManagerTimelock` | `uint256` | - | Sets cooldown duration |
| `addInstrRootGuardian` | external | `restricted` | `address` | - | Adds instruction root guardian |
| `removeInstrRootGuardian` | external | `restricted` | `address` | - | Removes instruction root guardian |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyOperator` | Only allows operator role to execute |
| `onlyRiskManager` | Only allows risk manager role to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `nonReentrant` | Prevents reentrancy attacks |
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 4. CaliberMailbox Contract

**File:** `makina-core/src/caliber/CaliberMailbox.sol`  
**Description:** Handles cross-chain communication and bridging for spoke calibers

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `MakinaGovernableInitParams, address` | - | Initializes the mailbox |
| `caliber` | external | `view` | - | `address` | Returns the caliber address |
| `getHubBridgeAdapter` | external | `view` | `uint16` | `address` | Returns hub bridge adapter for bridge ID |
| `getSpokeCaliberAccountingData` | external | `view` | - | `SpokeCaliberAccountingData` | Returns spoke caliber accounting data |
| `manageTransfer` | external | `nonReentrant` | `address, uint256, bytes` | - | Manages transfers between caliber and bridges |
| `sendOutBridgeTransfer` | external | `onlyOperator` | `uint16, uint256, bytes` | - | Sends outbound bridge transfer |
| `authorizeInBridgeTransfer` | external | `notRecoveryMode, onlyMechanic` | `uint16, bytes32` | - | Authorizes inbound bridge transfer |
| `claimInBridgeTransfer` | external | `onlyOperator` | `uint16, uint256` | - | Claims inbound bridge transfer |
| `cancelOutBridgeTransfer` | external | `onlyOperator` | `uint16, uint256` | - | Cancels outbound bridge transfer |
| `setCaliber` | external | `onlyFactory` | `address` | - | Sets the caliber address |
| `setHubBridgeAdapter` | external | `restricted` | `uint16, address` | - | Sets hub bridge adapter |
| `setOutTransferEnabled` | external | `onlyRiskManagerTimelock` | `uint16, bool` | - | Enables/disables outbound transfers |
| `setMaxBridgeLossBps` | external | `onlyRiskManagerTimelock` | `uint16, uint256` | - | Sets max bridge loss in basis points |
| `resetBridgingState` | external | `onlySecurityCouncil` | `address` | - | Resets bridging state for a token |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `onlyOperator` | Only allows operator role to execute |
| `onlyMechanic` | Only allows mechanic role to execute |
| `onlyFactory` | Only allows factory to execute |
| `onlyRiskManagerTimelock` | Only allows risk manager timelock to execute |
| `onlySecurityCouncil` | Only allows security council to execute |
| `nonReentrant` | Prevents reentrancy attacks |
| `notRecoveryMode` | Only allows execution when not in recovery mode |
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 5. BridgeController Contract

**File:** `makina-core/src/bridge/controller/BridgeController.sol`  
**Description:** Abstract contract managing bridge adapters and cross-chain transfers

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `isBridgeSupported` | external | `view` | `uint16` | `bool` | Checks if bridge is supported |
| `isOutTransferEnabled` | external | `view` | `uint16` | `bool` | Checks if outbound transfers are enabled |
| `getBridgeAdapter` | public | `view` | `uint16` | `address` | Returns bridge adapter address |
| `getMaxBridgeLossBps` | external | `view` | `uint16` | `uint256` | Returns max bridge loss in basis points |
| `createBridgeAdapter` | external | `restricted` | `uint16, uint256, bytes` | `address` | Creates a new bridge adapter |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `restricted` | General access control restriction |

---

## 6. HubCoreRegistry Contract

**File:** `makina-core/src/registries/HubCoreRegistry.sol`  
**Description:** Registry for core protocol components on the hub chain

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `address, address, address, address` | - | Initializes the registry |
| `chainRegistry` | external | `view` | - | `address` | Returns the chain registry address |
| `machineBeacon` | external | `view` | - | `address` | Returns the machine beacon address |
| `preDepositVaultBeacon` | external | `view` | - | `address` | Returns the pre-deposit vault beacon |
| `setChainRegistry` | external | `restricted` | `address` | - | Sets the chain registry address |
| `setMachineBeacon` | external | `restricted` | `address` | - | Sets the machine beacon address |
| `setPreDepositVaultBeacon` | external | `restricted` | `address` | - | Sets the pre-deposit vault beacon |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## 7. HubCoreFactory Contract

**File:** `makina-core/src/factories/HubCoreFactory.sol`  
**Description:** Factory for creating machines, calibers, and pre-deposit vaults

### Public Functions

| Function Name | Visibility | Modifiers | Parameters | Return Type | Description |
|---------------|------------|-----------|------------|-------------|-------------|
| `initialize` | external | `initializer` | `address` | - | Initializes the factory |
| `isMachine` | external | `view` | `address` | `bool` | Checks if address is a machine |
| `isPreDepositVault` | external | `view` | `address` | `bool` | Checks if address is a pre-deposit vault |
| `createPreDepositVault` | external | `restricted` | `PreDepositVaultInitParams, address, address, string, string` | `address` | Creates a pre-deposit vault |
| `createMachineFromPreDeposit` | external | `restricted` | `MachineInitParams, CaliberInitParams, MakinaGovernableInitParams, address, bytes32` | `address` | Creates machine from pre-deposit vault |
| `createMachine` | external | `restricted` | `MachineInitParams, CaliberInitParams, MakinaGovernableInitParams, address, string, string, bytes32` | `address` | Creates a new machine |
| `createBridgeAdapter` | external | - | `uint16, bytes` | `address` | Creates a bridge adapter |

### Modifiers

| Modifier Name | Description |
|---------------|-------------|
| `restricted` | General access control restriction |
| `initializer` | Ensures function can only be called once during initialization |

---

## Summary

The core contracts provide the fundamental infrastructure for:

1. **Asset Management**: Machine contract handles deposits, redemptions, and share management
2. **Strategy Execution**: Caliber contract executes complex strategies using Weiroll VM
3. **Cross-Chain Operations**: Bridge controllers and mailboxes handle multi-chain functionality
4. **Registry Management**: Registries track protocol components and configurations
5. **Factory Pattern**: Factories create and manage protocol instances

All contracts implement comprehensive access control with role-based permissions and include safety mechanisms like reentrancy protection and recovery modes.

