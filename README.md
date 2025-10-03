# Makina Contracts Documentation

Welcome to the Makina Contracts documentation repository. This directory contains comprehensive documentation for all interfaces and components in the Makina protocol.

## 📁 Documentation Files

### [INTERFACES.md](./INTERFACES.md)
**Complete Interface Enumeration for Makina Core and Periphery Contracts**

This document provides a detailed catalog of all 40 interfaces used across the Makina protocol, organized by functionality and repository.

**Contents:**
- **Makina Core Interfaces (26 interfaces)**
  - Core Registry & Context (4)
  - Factory Interfaces (4)
  - Machine & Caliber (4)
  - Bridge & Cross-Chain (6)
  - Oracle & Token Registry (3)
  - Fee Management (1)
  - Swap & Trading (1)
  - Governance & Access Control (2)
  - Utility Interfaces (1)

- **Makina Periphery Interfaces (14 interfaces)**
  - Periphery Registry & Context (2)
  - Periphery Factories (2)
  - Depositor & Redeemer (3)
  - Fee Management (1)
  - Security Module (3)
  - Oracle & Integration (1)
  - Flashloan (1)
  - Access Control (1)

## 🎯 Quick Reference

### Most Important Core Interfaces

| Interface | Purpose | Location |
|-----------|---------|----------|
| `IMachine` | Main vault managing assets across chains | `makina-core/src/interfaces/IMachine.sol` |
| `ICaliber` | Strategy execution vault with position management | `makina-core/src/interfaces/ICaliber.sol` |
| `IHubCoreRegistry` | Central registry for hub components | `makina-core/src/interfaces/IHubCoreRegistry.sol` |
| `IBridgeController` | Controls bridge operations | `makina-core/src/interfaces/IBridgeController.sol` |
| `IOracleRegistry` | Chainlink price feed aggregator | `makina-core/src/interfaces/IOracleRegistry.sol` |
| `IMakinaGovernable` | Governance and access control | `makina-core/src/interfaces/IMakinaGovernable.sol` |

### Most Important Periphery Interfaces

| Interface | Purpose | Location |
|-----------|---------|----------|
| `ISecurityModule` | Protocol insurance with slashing | `makina-periphery/src/interfaces/ISecurityModule.sol` |
| `IWatermarkFeeManager` | Fee management with high watermark | `makina-periphery/src/interfaces/IWatermarkFeeManager.sol` |
| `IAsyncRedeemer` | Asynchronous redemption system | `makina-periphery/src/interfaces/IAsyncRedeemer.sol` |
| `IFlashloanAggregator` | Multi-protocol flashloan aggregator | `makina-periphery/src/interfaces/IFlashloanAggregator.sol` |
| `IHubPeripheryFactory` | Factory for periphery components | `makina-periphery/src/interfaces/IHubPeripheryFactory.sol` |

## 🏗️ Architecture Overview

### Core Contracts
The core contracts handle the fundamental protocol operations:
- **Machine**: Cross-chain vault managing multiple calibers
- **Caliber**: Strategy execution vault with Weiroll-based instructions
- **Bridge System**: Unified interface for multiple bridge protocols
- **Registries**: Centralized configuration and component management

### Periphery Contracts
The periphery contracts provide extended functionality:
- **Depositors/Redeemers**: Flexible deposit and withdrawal mechanisms
- **Fee Managers**: Customizable fee calculation and distribution
- **Security Module**: Protocol insurance and slashing mechanism
- **Flashloan Aggregator**: Integration with multiple flashloan providers

## 🔑 Key Design Patterns

1. **Beacon Proxy Pattern**: Upgradeable deployments via beacon contracts
2. **Factory Pattern**: Standardized component deployment
3. **Registry Pattern**: Centralized component address management
4. **Modular Architecture**: Pluggable components (depositors, redeemers, fee managers)
5. **Multi-Signature Governance**: Separation of powers across different roles
6. **Bridge Abstraction**: Unified interface for cross-chain operations
7. **Instruction-Based Execution**: Flexible strategy execution via Weiroll

## 📊 Interface Inheritance Hierarchy

### Core Contracts
```
IMachineEndpoint
├── IBridgeController
└── IMakinaGovernable
    ├── IMachine (also inherits IMachineEndpoint)
    └── ICaliberMailbox (also inherits IMachineEndpoint)

ICoreRegistry
├── IHubCoreRegistry
└── ISpokeCoreRegistry

IBridgeAdapterFactory
├── IHubCoreFactory
└── ISpokeCoreFactory
```

### Periphery Contracts
```
IMachinePeriphery
├── IDirectDepositor
├── IAsyncRedeemer
├── ISecurityModule (also inherits IERC20Metadata)
└── IWatermarkFeeManager (also inherits IFeeManager, ISecurityModuleReference)

IERC721
└── ISMCooldownReceipt
```

## 🔍 Interface Usage Guide

### For Protocol Integrators

1. **To Deposit into a Machine:**
   - Use `IMachine.deposit()` directly, or
   - Use a depositor implementing `IDirectDepositor`

2. **To Redeem from a Machine:**
   - Use `IMachine.redeem()` directly, or
   - Use `IAsyncRedeemer` for asynchronous redemptions

3. **To Check Machine State:**
   - Query `IMachine` for vault-level data
   - Query `ICaliber` for strategy-level data
   - Query spoke calibers via `ICaliberMailbox`

4. **To Get Token Prices:**
   - Use `IOracleRegistry.getPrice()` for any token pair

5. **To Execute Cross-Chain Transfers:**
   - Call `IMachine.transferToSpokeCaliber()` or `transferToHubCaliber()`
   - Use `IBridgeController` for low-level bridge operations

### For Protocol Operators

1. **Managing Positions:**
   - Use `ICaliber.managePosition()` with Weiroll instructions
   - Monitor position freshness via `ICaliber.isAccountingFresh()`

2. **Managing Fees:**
   - Implement `IFeeManager` interface
   - Use `IWatermarkFeeManager` reference implementation

3. **Managing Security:**
   - Lock machine shares in `ISecurityModule`
   - Configure slashing parameters

4. **Managing Cross-Chain State:**
   - Update spoke caliber data via `IMachine.updateSpokeCaliberAccountingData()`
   - Configure bridge adapters via `IBridgeController`

## 📝 Version Information

- **Solidity Version:** 0.8.28
- **License:** MIT
- **Last Updated:** September 30, 2025

## 🔗 Related Resources

- **Makina Core Repository:** `makina-core/`
- **Makina Periphery Repository:** `makina-periphery/`
- **Interface Specifications:** [INTERFACES.md](./INTERFACES.md)

## 📧 Support

For questions or clarifications about the interfaces, please refer to the detailed interface documentation in [INTERFACES.md](./INTERFACES.md) or consult the inline documentation in the Solidity files.

---

**Note:** This documentation reflects the current state of the Makina protocol interfaces. Always refer to the source code for the most up-to-date information.
