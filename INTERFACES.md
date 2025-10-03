# Makina Contracts - Interface Documentation

This document provides a comprehensive enumeration of all important interfaces used in the Makina protocol, organized by repository (Core and Periphery) and categorized by functionality.

## Table of Contents
- [Makina Core Interfaces](#makina-core-interfaces)
  - [Core Registry & Context](#core-registry--context)
  - [Factory Interfaces](#factory-interfaces)
  - [Machine & Caliber](#machine--caliber)
  - [Bridge & Cross-Chain](#bridge--cross-chain)
  - [Oracle & Token Registry](#oracle--token-registry)
  - [Fee Management](#fee-management)
  - [Swap & Trading](#swap--trading)
  - [Governance & Access Control](#governance--access-control)
  - [Utility Interfaces](#utility-interfaces)
- [Makina Periphery Interfaces](#makina-periphery-interfaces)
  - [Periphery Registry & Context](#periphery-registry--context)
  - [Periphery Factories](#periphery-factories)
  - [Depositor & Redeemer](#depositor--redeemer)
  - [Fee Management (Periphery)](#fee-management-periphery)
  - [Security Module](#security-module)
  - [Oracle & Integration](#oracle--integration)
  - [Flashloan](#flashloan)
  - [Access Control (Periphery)](#access-control-periphery)

---

## Makina Core Interfaces

### Core Registry & Context

#### **ICoreRegistry**
**Location:** `makina-core/src/interfaces/ICoreRegistry.sol`

**Purpose:** Central registry for core protocol components.

**Key Functions:**
- `coreFactory()` - Returns the address of the core factory
- `oracleRegistry()` - Returns the oracle registry address
- `tokenRegistry()` - Returns the token registry address
- `swapModule()` - Returns the swap module address
- `flashLoanModule()` - Returns the flashloan module address
- `caliberBeacon()` - Returns the caliber beacon contract address
- `bridgeAdapterBeacon(uint16 bridgeId)` - Returns bridge adapter beacon for a specific bridge
- Setter functions for all registry components

**Events:**
- `BridgeAdapterBeaconChanged`
- `CaliberBeaconChanged`
- `CoreFactoryChanged`
- `FlashLoanModuleChanged`
- `OracleRegistryChanged`
- `SwapModuleChanged`
- `TokenRegistryChanged`

---

#### **IHubCoreRegistry**
**Location:** `makina-core/src/interfaces/IHubCoreRegistry.sol`

**Purpose:** Extended registry for hub-specific core components. Inherits from `ICoreRegistry`.

**Key Functions:**
- All functions from `ICoreRegistry`
- `chainRegistry()` - Returns the chain registry address
- `machineBeacon()` - Returns the machine beacon contract address
- `preDepositVaultBeacon()` - Returns the pre-deposit vault beacon address
- Setter functions for hub-specific components

**Events:**
- All events from `ICoreRegistry`
- `ChainRegistryChanged`
- `MachineBeaconChanged`
- `PreDepositVaultBeaconChanged`

---

#### **ISpokeCoreRegistry**
**Location:** `makina-core/src/interfaces/ISpokeCoreRegistry.sol`

**Purpose:** Extended registry for spoke chain core components. Inherits from `ICoreRegistry`.

**Key Functions:**
- All functions from `ICoreRegistry`
- `caliberMailboxBeacon()` - Returns the caliber mailbox beacon address
- `setCaliberMailboxBeacon(address)` - Sets the caliber mailbox beacon

**Events:**
- All events from `ICoreRegistry`
- `CaliberMailboxBeaconChanged`

---

#### **IMakinaContext**
**Location:** `makina-core/src/interfaces/IMakinaContext.sol`

**Purpose:** Provides context about the protocol registry.

**Key Functions:**
- `registry()` - Returns the address of the registry

---

### Factory Interfaces

#### **IHubCoreFactory**
**Location:** `makina-core/src/interfaces/IHubCoreFactory.sol`

**Purpose:** Factory for creating hub-side protocol components. Inherits from `IBridgeAdapterFactory`.

**Key Functions:**
- `isPreDepositVault(address)` - Checks if an address is a PreDepositVault deployed by this factory
- `isMachine(address)` - Checks if an address is a Machine deployed by this factory
- `createPreDepositVault(...)` - Deploys a new PreDepositVault
- `createMachineFromPreDeposit(...)` - Deploys a new Machine and migrates from PreDepositVault
- `createMachine(...)` - Deploys a new Machine instance
- `createBridgeAdapter(uint16, bytes)` - Creates a bridge adapter (from IBridgeAdapterFactory)

**Events:**
- `MachineCreated`
- `PreDepositVaultCreated`
- `ShareTokenCreated`
- `BridgeAdapterCreated` (from IBridgeAdapterFactory)

---

#### **ISpokeCoreFactory**
**Location:** `makina-core/src/interfaces/ISpokeCoreFactory.sol`

**Purpose:** Factory for creating spoke chain protocol components. Inherits from `IBridgeAdapterFactory`.

**Key Functions:**
- `isCaliberMailbox(address)` - Checks if an address is a CaliberMailbox deployed by this factory
- `createCaliber(...)` - Deploys a new Caliber instance
- `createBridgeAdapter(uint16, bytes)` - Creates a bridge adapter (from IBridgeAdapterFactory)

**Events:**
- `CaliberMailboxCreated`
- `BridgeAdapterCreated` (from IBridgeAdapterFactory)

---

#### **ICaliberFactory**
**Location:** `makina-core/src/interfaces/ICaliberFactory.sol`

**Purpose:** Factory interface for Caliber instances.

**Key Functions:**
- `isCaliber(address)` - Checks if an address is a Caliber deployed by this factory

**Events:**
- `CaliberCreated`

---

#### **IBridgeAdapterFactory**
**Location:** `makina-core/src/interfaces/IBridgeAdapterFactory.sol`

**Purpose:** Factory interface for creating bridge adapters.

**Key Functions:**
- `isBridgeAdapter(address)` - Checks if an address is a BridgeAdapter deployed by this factory
- `createBridgeAdapter(uint16, bytes)` - Deploys a bridge adapter instance

**Events:**
- `BridgeAdapterCreated`

---

### Machine & Caliber

#### **IMachine**
**Location:** `makina-core/src/interfaces/IMachine.sol`

**Purpose:** Core vault interface that manages assets across multiple calibers and chains. Inherits from `IMachineEndpoint`.

**Key Functions:**
- **Initialization:**
  - `initialize(...)` - Initializes the machine with parameters
- **State Queries:**
  - `wormhole()` - Returns Wormhole Core Bridge address
  - `depositor()`, `redeemer()`, `shareToken()`, `accountingToken()`, `hubCaliber()`, `feeManager()` - Component addresses
  - `caliberStaleThreshold()`, `maxFixedFeeAccrualRate()`, `maxPerfFeeAccrualRate()`, `feeMintCooldown()`, `shareLimit()` - Configuration parameters
  - `maxMint()`, `maxWithdraw()` - Maximum deposit/withdrawal limits
  - `lastTotalAum()`, `lastGlobalAccountingTime()` - Accounting state
  - `isIdleToken(address)` - Token classification
  - `getSpokeCalibersLength()`, `getSpokeChainId(uint256)` - Spoke caliber enumeration
  - `getSpokeCaliberDetailedAum(uint256)`, `getSpokeCaliberMailbox(uint256)`, `getSpokeBridgeAdapter(uint256, uint16)` - Spoke caliber data
- **Share Conversion:**
  - `convertToShares(uint256)` - Converts assets to shares
  - `convertToAssets(uint256)` - Converts shares to assets
- **Operations:**
  - `transferToHubCaliber(address, uint256)` - Transfers tokens to hub caliber
  - `transferToSpokeCaliber(uint16, uint256, address, uint256, uint256)` - Transfers tokens to spoke caliber
  - `updateTotalAum()` - Updates total AUM
  - `deposit(uint256, address, uint256)` - Deposits accounting tokens
  - `redeem(uint256, address, uint256)` - Redeems shares
  - `updateSpokeCaliberAccountingData(bytes, GuardianSignature[])` - Updates spoke caliber data using Wormhole CCQ
- **Configuration:**
  - `setSpokeCaliber(...)`, `setSpokeBridgeAdapter(...)` - Spoke caliber configuration
  - `setDepositor(address)`, `setRedeemer(address)`, `setFeeManager(address)` - Component updates
  - `setCaliberStaleThreshold(uint256)`, `setMaxFixedFeeAccrualRate(uint256)`, `setMaxPerfFeeAccrualRate(uint256)`, `setFeeMintCooldown(uint256)`, `setShareLimit(uint256)` - Parameter updates

**Structs:**
- `MachineInitParams` - Initialization parameters
- `SpokeCaliberData` - Spoke caliber internal state

**Events:**
- `CaliberStaleThresholdChanged`, `Deposit`, `DepositorChanged`, `FeeManagerChanged`, `FeeMintCooldownChanged`, `FeesMinted`, `MaxFixedFeeAccrualRateChanged`, `MaxPerfFeeAccrualRateChanged`, `Redeem`, `RedeemerChanged`, `ShareLimitChanged`, `SpokeBridgeAdapterSet`, `SpokeCaliberMailboxSet`, `TotalAumUpdated`, `TransferToCaliber`

---

#### **ICaliber**
**Location:** `makina-core/src/interfaces/ICaliber.sol`

**Purpose:** Strategy execution vault that manages positions and base tokens.

**Key Functions:**
- **Initialization:**
  - `initialize(...)` - Initializes the caliber
- **State Queries:**
  - `weirollVm()`, `hubMachineEndpoint()`, `accountingToken()` - Component addresses
  - `positionStaleThreshold()`, `allowedInstrRoot()`, `timelockDuration()`, `pendingAllowedInstrRoot()`, `pendingTimelockExpiry()` - Configuration
  - `maxPositionIncreaseLossBps()`, `maxPositionDecreaseLossBps()`, `maxSwapLossBps()`, `cooldownDuration()` - Loss tolerance and cooldown
  - `getPositionsLength()`, `getPositionId(uint256)`, `getPosition(uint256)` - Position enumeration
  - `isBaseToken(address)`, `getBaseTokensLength()`, `getBaseToken(uint256)` - Base token management
  - `isInstrRootGuardian(address)` - Guardian verification
  - `isAccountingFresh()` - Checks position accounting freshness
  - `getDetailedAum()` - Returns detailed AUM breakdown
- **Operations:**
  - `addBaseToken(address)`, `removeBaseToken(address)` - Base token management
  - `accountForPosition(Instruction)` - Accounts for a single position
  - `accountForPositionBatch(Instruction[], uint256[])` - Batch position accounting
  - `managePosition(Instruction, Instruction)` - Manages a position with paired instructions
  - `managePositionBatch(Instruction[], Instruction[])` - Batch position management
  - `manageFlashLoan(Instruction, address, uint256)` - Manages flashloan funds
  - `harvest(Instruction, SwapOrder[])` - Harvests positions
  - `swap(SwapOrder)` - Performs token swap
  - `transferToHubMachine(address, uint256, bytes)` - Transfers tokens to hub machine
  - `notifyIncomingTransfer(address, uint256)` - Handles incoming transfers
- **Configuration:**
  - `setPositionStaleThreshold(uint256)`, `setTimelockDuration(uint256)` - Accounting and timelock parameters
  - `scheduleAllowedInstrRootUpdate(bytes32)`, `cancelAllowedInstrRootUpdate()` - Merkle root updates
  - `setMaxPositionIncreaseLossBps(uint256)`, `setMaxPositionDecreaseLossBps(uint256)`, `setMaxSwapLossBps(uint256)` - Loss tolerance
  - `setCooldownDuration(uint256)` - Cooldown period
  - `addInstrRootGuardian(address)`, `removeInstrRootGuardian(address)` - Guardian management

**Structs:**
- `CaliberInitParams` - Initialization parameters
- `Instruction` - Instruction parameters for position management
- `Position` - Position data

**Enums:**
- `InstructionType` - Types of instructions (MANAGEMENT, ACCOUNTING, HARVEST, FLASHLOAN_MANAGEMENT)

**Events:**
- `BaseTokenAdded`, `BaseTokenRemoved`, `CooldownDurationChanged`, `IncomingTransfer`, `InstrRootGuardianAdded`, `InstrRootGuardianRemoved`, `MaxPositionDecreaseLossBpsChanged`, `MaxPositionIncreaseLossBpsChanged`, `MaxSwapLossBpsChanged`, `NewAllowedInstrRootCancelled`, `NewAllowedInstrRootScheduled`, `PositionClosed`, `PositionCreated`, `PositionUpdated`, `PositionStaleThresholdChanged`, `TimelockDurationChanged`, `TransferToHubMachine`

---

#### **IPreDepositVault**
**Location:** `makina-core/src/interfaces/IPreDepositVault.sol`

**Purpose:** Pre-launch vault for collecting deposits before machine deployment.

**Key Functions:**
- **Initialization:**
  - `initialize(...)` - Initializes the vault
- **State Queries:**
  - `migrated()` - Returns migration status
  - `machine()` - Returns machine address after migration
  - `riskManager()`, `whitelistMode()`, `isWhitelistedUser(address)` - Access control
  - `depositToken()`, `accountingToken()`, `shareToken()` - Token addresses
  - `shareLimit()`, `maxDeposit()`, `totalAssets()` - Capacity and assets
  - `previewDeposit(uint256)`, `previewRedeem(uint256)` - Preview conversions
- **Operations:**
  - `deposit(uint256, address, uint256)` - Deposits assets
  - `redeem(uint256, address, uint256)` - Redeems shares
  - `migrateToMachine()` - Migrates to machine
- **Configuration:**
  - `setPendingMachine(address)` - Sets pending machine for migration
  - `setRiskManager(address)`, `setShareLimit(uint256)` - Parameter updates
  - `setWhitelistedUsers(address[], bool)`, `setWhitelistMode(bool)` - Whitelist management

**Structs:**
- `PreDepositVaultInitParams` - Initialization parameters

**Events:**
- `Deposit`, `MigrateToMachine`, `Redeem`, `RiskManagerChanged`, `ShareLimitChanged`, `UserWhitelistingChanged`, `WhitelistModeChanged`

---

#### **IMachineShare**
**Location:** `makina-core/src/interfaces/IMachineShare.sol`

**Purpose:** Share token interface for machines. Inherits from `IERC20Metadata`.

**Key Functions:**
- All functions from `IERC20Metadata`
- `minter()` - Returns the authorized minter and burner address
- `mint(address, uint256)` - Mints new shares
- `burn(address, uint256)` - Burns shares

---

### Bridge & Cross-Chain

#### **IMachineEndpoint**
**Location:** `makina-core/src/interfaces/IMachineEndpoint.sol`

**Purpose:** Manages token transfers between machines and calibers. Inherits from `IBridgeController` and `IMakinaGovernable`.

**Key Functions:**
- All functions from `IBridgeController` and `IMakinaGovernable`
- `manageTransfer(address, uint256, bytes)` - Manages token transfers

---

#### **ICaliberMailbox**
**Location:** `makina-core/src/interfaces/ICaliberMailbox.sol`

**Purpose:** Spoke caliber endpoint for cross-chain communication. Inherits from `IMachineEndpoint`.

**Key Functions:**
- All functions from `IMachineEndpoint`
- `initialize(...)` - Initializes the mailbox
- `caliber()` - Returns associated caliber address
- `getHubBridgeAdapter(uint16)` - Returns hub bridge adapter for a bridge ID
- `hubChainId()` - Returns hub chain ID
- `getSpokeCaliberAccountingData()` - Returns caliber accounting data
- `setCaliber(address)` - Sets the caliber address
- `setHubBridgeAdapter(uint16, address)` - Registers a hub bridge adapter

**Structs:**
- `SpokeCaliberAccountingData` - Accounting data structure

**Events:**
- `CaliberSet`, `HubBridgeAdapterSet`

---

#### **IBridgeController**
**Location:** `makina-core/src/interfaces/IBridgeController.sol`

**Purpose:** Controls bridge operations and adapters.

**Key Functions:**
- `isBridgeSupported(uint16)` - Checks if a bridge is supported
- `isOutTransferEnabled(uint16)` - Checks if outgoing transfers are enabled for a bridge
- `getBridgeAdapter(uint16)` - Returns bridge adapter address
- `getMaxBridgeLossBps(uint16)` - Returns max allowed value loss for a bridge
- `createBridgeAdapter(uint16, uint256, bytes)` - Creates a bridge adapter
- `setMaxBridgeLossBps(uint16, uint256)` - Sets max bridge loss
- `setOutTransferEnabled(uint16, bool)` - Enables/disables outgoing transfers
- `sendOutBridgeTransfer(uint16, uint256, bytes)` - Executes outgoing bridge transfer
- `authorizeInBridgeTransfer(uint16, bytes32)` - Authorizes incoming bridge transfer
- `claimInBridgeTransfer(uint16, uint256)` - Claims received bridge transfer
- `cancelOutBridgeTransfer(uint16, uint256)` - Cancels outgoing bridge transfer
- `resetBridgingState(address)` - Resets bridge state for a token

**Events:**
- `BridgeAdapterCreated`, `MaxBridgeLossBpsChanged`, `BridgingStateReset`, `OutTransferEnabledSet`

---

#### **IBridgeAdapter**
**Location:** `makina-core/src/interfaces/IBridgeAdapter.sol`

**Purpose:** Adapter for external bridge protocols.

**Key Functions:**
- `initialize(address, bytes)` - Initializes the adapter
- `controller()` - Returns bridge controller address
- `bridgeId()` - Returns external bridge ID
- `approvalTarget()`, `executionTarget()`, `receiveSource()` - External bridge contract addresses
- `nextOutTransferId()`, `nextInTransferId()` - Transfer ID counters
- `scheduleOutBridgeTransfer(...)` - Schedules an outgoing transfer
- `sendOutBridgeTransfer(uint256, bytes)` - Executes a scheduled transfer
- `outBridgeTransferCancelDefault(uint256)` - Returns cancel cost
- `cancelOutBridgeTransfer(uint256)` - Cancels a transfer
- `authorizeInBridgeTransfer(bytes32)` - Authorizes incoming transfer
- `claimInBridgeTransfer(uint256)` - Claims received transfer
- `withdrawPendingFunds(address)` - Withdraws stuck funds

**Structs:**
- `OutBridgeTransfer` - Outgoing transfer data
- `InBridgeTransfer` - Incoming transfer data
- `BridgeMessage` - Bridge message data

**Events:**
- `InBridgeTransferAuthorized`, `OutBridgeTransferCancelled`, `InBridgeTransferClaimed`, `InBridgeTransferReceived`, `OutBridgeTransferSent`, `OutBridgeTransferScheduled`, `PendingFundsWithdrawn`

---

#### **IAcrossV3SpokePool**
**Location:** `makina-core/src/interfaces/IAcrossV3SpokePool.sol`

**Purpose:** External interface for Across V3 bridge integration.

**Key Functions:**
- `depositV3Now(...)` - Initiates an Across V3 deposit

---

#### **IAcrossV3MessageHandler**
**Location:** `makina-core/src/interfaces/IAcrossV3MessageHandler.sol`

**Purpose:** Handler for Across V3 cross-chain messages.

**Key Functions:**
- `handleV3AcrossMessage(address, uint256, address, bytes)` - Handles Across V3 messages

---

### Oracle & Token Registry

#### **IOracleRegistry**
**Location:** `makina-core/src/interfaces/IOracleRegistry.sol`

**Purpose:** Aggregates Chainlink price feeds for token pricing.

**Key Functions:**
- `getFeedStaleThreshold(address)` - Returns staleness threshold for a feed
- `isFeedRouteRegistered(address)` - Checks if a feed route is registered
- `getFeedRoute(address)` - Returns price feed route for a token
- `getPrice(address, address)` - Returns price of baseToken in terms of quoteToken
- `setFeedRoute(address, address, uint256, address, uint256)` - Sets price feed route
- `setFeedStaleThreshold(address, uint256)` - Sets staleness threshold

**Structs:**
- `FeedRoute` - Price feed route structure

**Events:**
- `FeedRouteRegistered`, `FeedStaleThresholdChanged`

---

#### **ITokenRegistry**
**Location:** `makina-core/src/interfaces/ITokenRegistry.sol`

**Purpose:** Maps token addresses across EVM chains.

**Key Functions:**
- `getForeignToken(address, uint256)` - Returns foreign token address
- `getLocalToken(address, uint256)` - Returns local token address
- `setToken(address, uint256, address)` - Associates local and foreign tokens

**Events:**
- `TokenRegistered`

---

#### **IChainRegistry**
**Location:** `makina-core/src/interfaces/IChainRegistry.sol`

**Purpose:** Maps EVM chain IDs to Wormhole chain IDs.

**Key Functions:**
- `isEvmChainIdRegistered(uint256)` - Checks if EVM chain ID is registered
- `isWhChainIdRegistered(uint16)` - Checks if Wormhole chain ID is registered
- `evmToWhChainId(uint256)` - Converts EVM to Wormhole chain ID
- `whToEvmChainId(uint16)` - Converts Wormhole to EVM chain ID
- `setChainIds(uint256, uint16)` - Associates chain IDs

**Events:**
- `ChainIdsRegistered`

---

### Fee Management

#### **IFeeManager**
**Location:** `makina-core/src/interfaces/IFeeManager.sol`

**Purpose:** Calculates and distributes fees for machines.

**Key Functions:**
- `calculateFixedFee(uint256, uint256)` - Calculates fixed fee
- `calculatePerformanceFee(uint256, uint256, uint256, uint256)` - Calculates performance fee
- `distributeFees(uint256, uint256)` - Distributes fees to recipients

---

### Swap & Trading

#### **ISwapModule**
**Location:** `makina-core/src/interfaces/ISwapModule.sol`

**Purpose:** Handles token swaps via external swap protocols.

**Key Functions:**
- `getSwapperTargets(uint16)` - Returns approval and execution targets for a swapper
- `swap(SwapOrder)` - Executes a token swap
- `setSwapperTargets(uint16, address, address)` - Sets swapper targets

**Structs:**
- `SwapperTargets` - Swapper target addresses
- `SwapOrder` - Swap order parameters

**Events:**
- `Swap`, `SwapperTargetsSet`

---

### Governance & Access Control

#### **IMakinaGovernable**
**Location:** `makina-core/src/interfaces/IMakinaGovernable.sol`

**Purpose:** Governance and access control for protocol components.

**Key Functions:**
- `mechanic()`, `securityCouncil()`, `riskManager()`, `riskManagerTimelock()` - Returns governance addresses
- `recoveryMode()` - Returns recovery mode status
- `setMechanic(address)`, `setSecurityCouncil(address)`, `setRiskManager(address)`, `setRiskManagerTimelock(address)` - Sets governance addresses
- `setRecoveryMode(bool)` - Enables/disables recovery mode

**Structs:**
- `MakinaGovernableInitParams` - Initialization parameters

**Events:**
- `MechanicChanged`, `RecoveryModeChanged`, `RiskManagerChanged`, `RiskManagerTimelockChanged`, `SecurityCouncilChanged`

---

#### **IOwnable2Step**
**Location:** `makina-core/src/interfaces/IOwnable2Step.sol`

**Purpose:** Two-step ownership transfer pattern.

**Key Functions:**
- `owner()` - Returns current owner
- `pendingOwner()` - Returns pending owner
- `transferOwnership(address)` - Initiates ownership transfer
- `acceptOwnership()` - Accepts ownership transfer

---

### Utility Interfaces

#### **IWeirollVM**
**Location:** `makina-core/src/interfaces/IWeirollVM.sol`

**Purpose:** Executes Weiroll commands for position management.

**Key Functions:**
- `execute(bytes32[], bytes[])` - Executes a list of commands

---

## Makina Periphery Interfaces

### Periphery Registry & Context

#### **IHubPeripheryRegistry**
**Location:** `makina-periphery/src/interfaces/IHubPeripheryRegistry.sol`

**Purpose:** Central registry for periphery components.

**Key Functions:**
- `peripheryFactory()` - Returns periphery factory address
- `depositorBeacon(uint16)` - Returns depositor beacon for an implementation ID
- `redeemerBeacon(uint16)` - Returns redeemer beacon for an implementation ID
- `feeManagerBeacon(uint16)` - Returns fee manager beacon for an implementation ID
- `securityModuleBeacon()` - Returns security module beacon address
- `setPeripheryFactory(address)` - Sets periphery factory
- `setDepositorBeacon(uint16, address)`, `setRedeemerBeacon(uint16, address)`, `setFeeManagerBeacon(uint16, address)` - Sets component beacons
- `setSecurityModuleBeacon(address)` - Sets security module beacon

**Events:**
- `DepositorBeaconChanged`, `FeeManagerBeaconChanged`, `PeripheryFactoryChanged`, `RedeemerBeaconChanged`, `SecurityModuleBeaconChanged`

---

#### **IMakinaPeripheryContext**
**Location:** `makina-periphery/src/interfaces/IMakinaPeripheryContext.sol`

**Purpose:** Provides context about the periphery registry.

**Key Functions:**
- `peripheryRegistry()` - Returns periphery registry address

---

### Periphery Factories

#### **IHubPeripheryFactory**
**Location:** `makina-periphery/src/interfaces/IHubPeripheryFactory.sol`

**Purpose:** Factory for creating periphery components.

**Key Functions:**
- `isDepositor(address)`, `isRedeemer(address)`, `isFeeManager(address)`, `isSecurityModule(address)` - Verification functions
- `depositorImplemId(address)`, `redeemerImplemId(address)`, `feeManagerImplemId(address)` - Returns implementation IDs
- `setMachine(address, address)` - Sets machine in periphery contract
- `setSecurityModule(address, address)` - Sets security module in fee manager
- `createDepositor(uint16, bytes)` - Creates a depositor
- `createRedeemer(uint16, bytes)` - Creates a redeemer
- `createFeeManager(uint16, bytes)` - Creates a fee manager
- `createSecurityModule(SecurityModuleInitParams)` - Creates a security module

**Events:**
- `DepositorCreated`, `RedeemerCreated`, `FeeManagerCreated`, `SecurityModuleCreated`

---

#### **IMetaMorphoOracleFactory**
**Location:** `makina-periphery/src/interfaces/IMetaMorphoOracleFactory.sol`

**Purpose:** Factory for creating MetaMorpho oracle wrappers.

**Key Functions:**
- `isMorphoFactory(address)` - Checks if a Morpho factory is trusted
- `isOracle(address)` - Checks if an oracle was deployed by this factory
- `setMorphoFactory(address, bool)` - Sets Morpho factory trust status
- `createMetaMorphoOracle(address, address, uint8)` - Creates a MetaMorpho oracle

**Events:**
- `MetaMorphoOracleCreated`

**Errors:**
- `NotMetaMorphoVault`, `NotFactory`

---

### Depositor & Redeemer

#### **IDirectDepositor**
**Location:** `makina-periphery/src/interfaces/IDirectDepositor.sol`

**Purpose:** Direct depositor implementation. Inherits from `IMachinePeriphery`.

**Key Functions:**
- All functions from `IMachinePeriphery`
- `deposit(uint256, address, uint256)` - Deposits assets to machine

---

#### **IAsyncRedeemer**
**Location:** `makina-periphery/src/interfaces/IAsyncRedeemer.sol`

**Purpose:** Asynchronous redemption with request/finalize pattern. Inherits from `IMachinePeriphery`.

**Key Functions:**
- All functions from `IMachinePeriphery`
- `nextRequestId()`, `lastFinalizedRequestId()` - Request ID tracking
- `finalizationDelay()` - Returns minimum delay before finalization
- `getShares(uint256)`, `getClaimableAssets(uint256)` - Request data queries
- `previewFinalizeRequests(uint256)` - Previews batch finalization
- `requestRedeem(uint256, address)` - Creates a redeem request
- `finalizeRequests(uint256, uint256)` - Finalizes pending requests
- `claimAssets(uint256)` - Claims finalized assets
- `setFinalizationDelay(uint256)` - Sets finalization delay

**Structs:**
- `RedeemRequest` - Redemption request data

**Events:**
- `FinalizationDelayChanged`, `RedeemRequestCreated`, `RedeemRequestClaimed`, `RedeemRequestsFinalized`

---

#### **IMachinePeriphery**
**Location:** `makina-periphery/src/interfaces/IMachinePeriphery.sol`

**Purpose:** Base interface for machine periphery contracts.

**Key Functions:**
- `initialize(bytes)` - Initializes the contract
- `machine()` - Returns associated machine address
- `setMachine(address)` - Sets the machine address

**Events:**
- `MachineSet`

---

### Fee Management (Periphery)

#### **IWatermarkFeeManager**
**Location:** `makina-periphery/src/interfaces/IWatermarkFeeManager.sol`

**Purpose:** Fee manager with high watermark for performance fees. Inherits from `IFeeManager`, `ISecurityModuleReference`, and `IMachinePeriphery`.

**Key Functions:**
- All functions from `IFeeManager`, `ISecurityModuleReference`, and `IMachinePeriphery`
- `mgmtFeeRatePerSecond()`, `smFeeRatePerSecond()`, `perfFeeRate()` - Fee rates
- `mgmtFeeReceivers()`, `mgmtFeeSplitBps()` - Management fee split configuration
- `perfFeeReceivers()`, `perfFeeSplitBps()` - Performance fee split configuration
- `sharePriceWatermark()` - Current high watermark
- `resetSharePriceWatermark(uint256)` - Resets watermark
- `setMgmtFeeRatePerSecond(uint256)`, `setSmFeeRatePerSecond(uint256)`, `setPerfFeeRate(uint256)` - Sets fee rates
- `setMgmtFeeSplit(address[], uint256[])`, `setPerfFeeSplit(address[], uint256[])` - Sets fee splits

**Structs:**
- `WatermarkFeeManagerInitParams` - Initialization parameters

**Events:**
- `MgmtFeeSplitChanged`, `MgmtFeeRatePerSecondChanged`, `PerfFeeRateChanged`, `PerfFeeSplitChanged`, `SmFeeRatePerSecondChanged`, `SecurityModuleSet`, `WatermarkReset`

---

### Security Module

#### **ISecurityModule**
**Location:** `makina-periphery/src/interfaces/ISecurityModule.sol`

**Purpose:** Security module for protocol insurance with slashing capability. Inherits from `IERC20Metadata` and `IMachinePeriphery`.

**Key Functions:**
- All functions from `IERC20Metadata` and `IMachinePeriphery`
- `machineShare()` - Returns machine share token address
- `cooldownReceipt()` - Returns cooldown receipt NFT address
- `cooldownDuration()`, `maxSlashableBps()`, `minBalanceAfterSlash()` - Configuration parameters
- `pendingCooldown(uint256)` - Returns pending cooldown data
- `slashingMode()` - Returns slashing mode status
- `totalLockedAmount()`, `maxSlashable()` - Locked amount queries
- `convertToShares(uint256)`, `convertToAssets(uint256)` - Share conversions
- `previewLock(uint256)` - Previews lock operation
- `lock(uint256, address, uint256)` - Locks machine shares
- `startCooldown(uint256, address)` - Starts unlock cooldown
- `cancelCooldown(uint256)` - Cancels a cooldown
- `redeem(uint256, uint256)` - Redeems after cooldown
- `slash(uint256)` - Slashes locked funds
- `settleSlashing()` - Settles slashing
- `setCooldownDuration(uint256)`, `setMaxSlashableBps(uint256)`, `setMinBalanceAfterSlash(uint256)` - Parameter updates

**Structs:**
- `SecurityModuleInitParams` - Initialization parameters
- `PendingCooldown` - Cooldown data

**Events:**
- `Cooldown`, `CooldownCancelled`, `CooldownDurationChanged`, `MaxSlashableBpsChanged`, `MinBalanceAfterSlashChanged`, `Lock`, `Redeem`, `Slash`, `SlashingSettled`

---

#### **ISecurityModuleReference**
**Location:** `makina-periphery/src/interfaces/ISecurityModuleReference.sol`

**Purpose:** Reference to a security module.

**Key Functions:**
- `securityModule()` - Returns security module address
- `setSecurityModule(address)` - Sets security module

---

#### **ISMCooldownReceipt**
**Location:** `makina-periphery/src/interfaces/ISMCooldownReceipt.sol`

**Purpose:** NFT receipt for security module cooldowns. Inherits from `IERC721`.

**Key Functions:**
- All functions from `IERC721`
- `nextTokenId()` - Returns next token ID
- `mint(address)` - Mints a cooldown receipt NFT
- `burn(uint256)` - Burns a cooldown receipt NFT

---

### Oracle & Integration

#### **IMetaMorphoFactory**
**Location:** `makina-periphery/src/interfaces/IMetaMorphoFactory.sol`

**Purpose:** External interface for MetaMorpho factory.

**Key Functions:**
- `isMetaMorpho(address)` - Checks if an address is a MetaMorpho vault

---

### Flashloan

#### **IFlashloanAggregator**
**Location:** `makina-periphery/src/interfaces/IFlashloanAggregator.sol`

**Purpose:** Aggregates multiple flashloan providers (Aave V3, Balancer V2/V3, Morpho, Maker DSS Flash).

**Key Functions:**
- `requestFlashloan(FlashloanRequest)` - Requests a flashloan

**Structs:**
- `FlashloanRequest` - Flashloan request parameters

**Enums:**
- `FlashloanProvider` - Supported flashloan providers (AAVE_V3, BALANCER_V2, BALANCER_V3, MORPHO, DSS_FLASH)

**Errors:**
- `NotCaliber`, `NotRequested`, `InvalidToken`, `InvalidParamsLength`, `InvalidFeeAmount`, `NotBalancerV2Pool`, `NotBalancerV3Pool`, `NotMorpho`, `NotDssFlash`, `NotAaveV3Pool`, `BalancerV2PoolNotSet`, `BalancerV3PoolNotSet`, `MorphoPoolNotSet`, `DssFlashNotSet`, `AaveV3PoolNotSet`, `InvalidUserDataHash`

---

### Access Control (Periphery)

#### **IWhitelist**
**Location:** `makina-periphery/src/interfaces/IWhitelist.sol`

**Purpose:** Whitelist functionality for access control.

**Key Functions:**
- `isWhitelistEnabled()` - Returns whitelist status
- `isWhitelistedUser(address)` - Checks if a user is whitelisted
- `setWhitelistStatus(bool)` - Enables/disables whitelist
- `setWhitelistedUsers(address[], bool)` - Updates user whitelist status

**Events:**
- `UserWhitelistingChanged`, `WhitelistStatusChanged`

---

## Summary Statistics

### Makina Core
- **Total Interfaces:** 26
- **Categories:**
  - Registry & Context: 4
  - Factories: 4
  - Machine & Caliber: 4
  - Bridge & Cross-Chain: 6
  - Oracle & Registry: 3
  - Fee Management: 1
  - Swap: 1
  - Governance: 2
  - Utility: 1

### Makina Periphery
- **Total Interfaces:** 14
- **Categories:**
  - Registry & Context: 2
  - Factories: 2
  - Depositor & Redeemer: 3
  - Fee Management: 1
  - Security Module: 3
  - Oracle Integration: 1
  - Flashloan: 1
  - Access Control: 1

### Total Interfaces: 40

---

## Key Design Patterns

### 1. **Beacon Proxy Pattern**
Used extensively for upgradeable deployments via beacons registered in registries (e.g., `machineBeacon`, `caliberBeacon`, `depositorBeacon`).

### 2. **Factory Pattern**
All major components are deployed through factories (`IHubCoreFactory`, `ISpokeCoreFactory`, `IHubPeripheryFactory`).

### 3. **Registry Pattern**
Centralized registries (`ICoreRegistry`, `IHubPeripheryRegistry`) manage component addresses and configurations.

### 4. **Two-Step Ownership Transfer**
Implemented via `IOwnable2Step` for safe ownership transfers.

### 5. **Governance Separation**
Multiple governance roles (`mechanic`, `securityCouncil`, `riskManager`, `riskManagerTimelock`) via `IMakinaGovernable`.

### 6. **Modular Fee Management**
Pluggable fee managers implementing `IFeeManager` interface.

### 7. **Bridge Abstraction**
Unified bridge interface via `IBridgeAdapter` supporting multiple bridge protocols.

### 8. **Instruction-Based Position Management**
Weiroll-based instruction execution for flexible position strategies.

---

**Document Version:** 1.0  
**Last Updated:** September 30, 2025  
**Repository:** Makina Contracts (Core + Periphery)
