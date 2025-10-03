# Interface Quick Reference Table

This document provides a quick reference table for all interfaces in the Makina protocol.

## Makina Core Interfaces (26 Total)

| # | Interface Name | Category | File Path | Key Purpose |
|---|----------------|----------|-----------|-------------|
| 1 | `ICoreRegistry` | Registry | `makina-core/src/interfaces/ICoreRegistry.sol` | Central registry for core components |
| 2 | `IHubCoreRegistry` | Registry | `makina-core/src/interfaces/IHubCoreRegistry.sol` | Hub-specific core registry (extends ICoreRegistry) |
| 3 | `ISpokeCoreRegistry` | Registry | `makina-core/src/interfaces/ISpokeCoreRegistry.sol` | Spoke-specific core registry (extends ICoreRegistry) |
| 4 | `IMakinaContext` | Context | `makina-core/src/interfaces/IMakinaContext.sol` | Provides registry context |
| 5 | `IHubCoreFactory` | Factory | `makina-core/src/interfaces/IHubCoreFactory.sol` | Creates machines, pre-deposit vaults, bridge adapters |
| 6 | `ISpokeCoreFactory` | Factory | `makina-core/src/interfaces/ISpokeCoreFactory.sol` | Creates calibers, mailboxes, bridge adapters |
| 7 | `ICaliberFactory` | Factory | `makina-core/src/interfaces/ICaliberFactory.sol` | Caliber factory verification |
| 8 | `IBridgeAdapterFactory` | Factory | `makina-core/src/interfaces/IBridgeAdapterFactory.sol` | Creates bridge adapters |
| 9 | `IMachine` | Core Vault | `makina-core/src/interfaces/IMachine.sol` | Main cross-chain vault |
| 10 | `ICaliber` | Strategy Vault | `makina-core/src/interfaces/ICaliber.sol` | Strategy execution vault with positions |
| 11 | `IPreDepositVault` | Vault | `makina-core/src/interfaces/IPreDepositVault.sol` | Pre-launch deposit collection |
| 12 | `IMachineShare` | Token | `makina-core/src/interfaces/IMachineShare.sol` | Machine share token (ERC20) |
| 13 | `IMachineEndpoint` | Bridge | `makina-core/src/interfaces/IMachineEndpoint.sol` | Token transfer management |
| 14 | `ICaliberMailbox` | Bridge | `makina-core/src/interfaces/ICaliberMailbox.sol` | Spoke caliber cross-chain endpoint |
| 15 | `IBridgeController` | Bridge | `makina-core/src/interfaces/IBridgeController.sol` | Bridge operations controller |
| 16 | `IBridgeAdapter` | Bridge | `makina-core/src/interfaces/IBridgeAdapter.sol` | External bridge protocol adapter |
| 17 | `IAcrossV3SpokePool` | Bridge | `makina-core/src/interfaces/IAcrossV3SpokePool.sol` | Across V3 bridge interface |
| 18 | `IAcrossV3MessageHandler` | Bridge | `makina-core/src/interfaces/IAcrossV3MessageHandler.sol` | Across V3 message handler |
| 19 | `IOracleRegistry` | Oracle | `makina-core/src/interfaces/IOracleRegistry.sol` | Chainlink price feed aggregator |
| 20 | `ITokenRegistry` | Registry | `makina-core/src/interfaces/ITokenRegistry.sol` | Cross-chain token address mapping |
| 21 | `IChainRegistry` | Registry | `makina-core/src/interfaces/IChainRegistry.sol` | EVM ↔ Wormhole chain ID mapping |
| 22 | `IFeeManager` | Fee | `makina-core/src/interfaces/IFeeManager.sol` | Fee calculation and distribution |
| 23 | `ISwapModule` | Swap | `makina-core/src/interfaces/ISwapModule.sol` | Token swap via external protocols |
| 24 | `IMakinaGovernable` | Governance | `makina-core/src/interfaces/IMakinaGovernable.sol` | Multi-role governance |
| 25 | `IOwnable2Step` | Governance | `makina-core/src/interfaces/IOwnable2Step.sol` | Two-step ownership transfer |
| 26 | `IWeirollVM` | Utility | `makina-core/src/interfaces/IWeirollVM.sol` | Weiroll command execution |

## Makina Periphery Interfaces (14 Total)

| # | Interface Name | Category | File Path | Key Purpose |
|---|----------------|----------|-----------|-------------|
| 1 | `IHubPeripheryRegistry` | Registry | `makina-periphery/src/interfaces/IHubPeripheryRegistry.sol` | Central periphery component registry |
| 2 | `IMakinaPeripheryContext` | Context | `makina-periphery/src/interfaces/IMakinaPeripheryContext.sol` | Provides periphery registry context |
| 3 | `IHubPeripheryFactory` | Factory | `makina-periphery/src/interfaces/IHubPeripheryFactory.sol` | Creates depositors, redeemers, fee managers |
| 4 | `IMetaMorphoOracleFactory` | Factory | `makina-periphery/src/interfaces/IMetaMorphoOracleFactory.sol` | Creates MetaMorpho oracle wrappers |
| 5 | `IMachinePeriphery` | Base | `makina-periphery/src/interfaces/IMachinePeriphery.sol` | Base interface for machine peripherals |
| 6 | `IDirectDepositor` | Depositor | `makina-periphery/src/interfaces/IDirectDepositor.sol` | Direct deposit implementation |
| 7 | `IAsyncRedeemer` | Redeemer | `makina-periphery/src/interfaces/IAsyncRedeemer.sol` | Asynchronous redemption system |
| 8 | `IWatermarkFeeManager` | Fee | `makina-periphery/src/interfaces/IWatermarkFeeManager.sol` | High watermark fee manager |
| 9 | `ISecurityModule` | Security | `makina-periphery/src/interfaces/ISecurityModule.sol` | Protocol insurance with slashing |
| 10 | `ISecurityModuleReference` | Security | `makina-periphery/src/interfaces/ISecurityModuleReference.sol` | Security module reference holder |
| 11 | `ISMCooldownReceipt` | Security | `makina-periphery/src/interfaces/ISMCooldownReceipt.sol` | Cooldown NFT receipt (ERC721) |
| 12 | `IMetaMorphoFactory` | Integration | `makina-periphery/src/interfaces/IMetaMorphoFactory.sol` | MetaMorpho factory interface |
| 13 | `IFlashloanAggregator` | Flashloan | `makina-periphery/src/interfaces/IFlashloanAggregator.sol` | Multi-protocol flashloan aggregator |
| 14 | `IWhitelist` | Access | `makina-periphery/src/interfaces/IWhitelist.sol` | Whitelist access control |

## Interface Dependency Graph

### Core Dependencies
```
IMakinaGovernable
    ↓
IMachineEndpoint (+ IBridgeController)
    ↓
┌────────────┬──────────────┐
│            │              │
IMachine  ICaliber  ICaliberMailbox
```

### Periphery Dependencies
```
IMachinePeriphery
    ↓
┌────────────┬──────────────┬────────────────┐
│            │              │                │
IDirectDepositor  IAsyncRedeemer  ISecurityModule  IWatermarkFeeManager
                                                        ↓
                                                  IFeeManager
                                                  ISecurityModuleReference
```

## Interface Usage by Role

### Protocol Users (Investors)
- `IMachine` - Deposit/redeem
- `IPreDepositVault` - Early deposit
- `IDirectDepositor` - Direct deposits
- `IAsyncRedeemer` - Asynchronous redemptions
- `ISecurityModule` - Lock/unlock security shares

### Protocol Operators (Mechanics)
- `ICaliber` - Manage positions
- `IMachine` - Update AUM, cross-chain transfers
- `IBridgeController` - Execute bridge operations
- `ISwapModule` - Execute swaps
- `IAsyncRedeemer` - Finalize redemptions

### Protocol Governors (DAO/Security Council)
- `IMakinaGovernable` - Update roles, recovery mode
- Registry interfaces - Update component addresses
- Factory interfaces - Deploy new components
- `ISecurityModule` - Slash protocol insurance

### Protocol Deployers
- `IHubCoreFactory` - Deploy machines, vaults
- `ISpokeCoreFactory` - Deploy calibers, mailboxes
- `IHubPeripheryFactory` - Deploy periphery components
- `IBridgeAdapterFactory` - Deploy bridge adapters

## Cross-Chain Interfaces

### Hub Chain Only
- `IMachine`
- `IPreDepositVault`
- `IHubCoreRegistry`
- `IHubCoreFactory`
- All periphery interfaces

### Spoke Chains Only
- `ICaliberMailbox`
- `ISpokeCoreRegistry`
- `ISpokeCoreFactory`

### Both Hub and Spoke
- `ICaliber`
- `IBridgeAdapter`
- `IBridgeController`
- `IMakinaGovernable`
- `ISwapModule`
- `IOracleRegistry`

## Interface Relationship Matrix

| Interface | Inherits From | Used By | Depends On |
|-----------|--------------|---------|------------|
| `IMachine` | `IMachineEndpoint` | Depositors, Redeemers, Fee Managers | `IMachineShare`, `ICaliber` |
| `ICaliber` | `ISwapModule` | `IMachine`, `ICaliberMailbox` | `IWeirollVM`, `ISwapModule` |
| `IMachineEndpoint` | `IBridgeController`, `IMakinaGovernable` | `IMachine`, `ICaliberMailbox` | `IBridgeAdapter` |
| `IWatermarkFeeManager` | `IFeeManager`, `IMachinePeriphery`, `ISecurityModuleReference` | `IMachine` | `ISecurityModule` |
| `ISecurityModule` | `IERC20Metadata`, `IMachinePeriphery` | `IWatermarkFeeManager` | `IMachineShare`, `ISMCooldownReceipt` |
| `IAsyncRedeemer` | `IMachinePeriphery` | Users | `IMachine` |

## Integration Interfaces (External Protocols)

| Interface | Protocol | Purpose |
|-----------|----------|---------|
| `IAcrossV3SpokePool` | Across Protocol | Bridge integration |
| `IAcrossV3MessageHandler` | Across Protocol | Cross-chain messaging |
| `IMetaMorphoFactory` | Morpho | Vault integration |
| `IMetaMorphoOracleFactory` | Morpho | Oracle wrapper creation |
| `IFlashloanAggregator` | Multiple (Aave, Balancer, Morpho, Maker) | Flashloan aggregation |
| (via `ISwapModule`) | Multiple DEXs | Token swapping |
| (via `IOracleRegistry`) | Chainlink | Price feeds |

## Total Count Summary

- **Total Interfaces:** 40
- **Core Interfaces:** 26
- **Periphery Interfaces:** 14
- **Factory Interfaces:** 6
- **Registry Interfaces:** 5
- **Governance Interfaces:** 2
- **Bridge Interfaces:** 6
- **Token Interfaces:** 3 (IMachineShare, ISecurityModule, ISMCooldownReceipt)

---

**Last Updated:** September 30, 2025  
**Document Version:** 1.0
