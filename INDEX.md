# Makina Contracts Documentation Index

Welcome to the Makina Contracts documentation. This index provides quick navigation to all documentation resources.

## 📚 Documentation Files

### 1. [README.md](./README.md) - **START HERE**
Overview and quick start guide for the documentation.
- Quick reference tables
- Most important interfaces
- Architecture overview
- Usage guide for different roles

### 2. [INTERFACES.md](./INTERFACES.md) - **MAIN REFERENCE** ⭐
Complete enumeration of all 40 interfaces with detailed specifications.
- **873 lines** of comprehensive documentation
- All function signatures
- All events and structs
- Inheritance relationships
- Usage examples

**Sections:**
- Makina Core Interfaces (26 interfaces)
- Makina Periphery Interfaces (14 interfaces)
- Summary statistics
- Key design patterns

### 3. [INTERFACE_QUICK_REFERENCE.md](./INTERFACE_QUICK_REFERENCE.md) - **QUICK LOOKUP**
Fast reference tables for all interfaces.
- Interface summary tables
- Dependency graphs
- Role-based usage guide
- Cross-chain deployment matrix
- Integration interface list

### 4. [ARCHITECTURE_DIAGRAM.md](./ARCHITECTURE_DIAGRAM.md) - **VISUAL GUIDE**
Visual diagrams and flow charts.
- High-level architecture
- Hub chain architecture
- Spoke chain architecture
- Component diagrams
- Data flow diagrams
- Lifecycle diagrams

## 🎯 Quick Navigation by Topic

### For Protocol Users
- [How to Deposit](./README.md#-interface-usage-guide) → `IMachine`, `IDirectDepositor`
- [How to Redeem](./README.md#-interface-usage-guide) → `IMachine`, `IAsyncRedeemer`
- [Security Module](./INTERFACES.md#isecuritymodule) → `ISecurityModule`

### For Protocol Operators
- [Managing Positions](./INTERFACES.md#icaliber) → `ICaliber`
- [Cross-Chain Operations](./INTERFACES.md#bridge--cross-chain) → `IMachine`, `IBridgeController`
- [Fee Management](./INTERFACES.md#ifeemanager) → `IFeeManager`, `IWatermarkFeeManager`

### For Protocol Governors
- [Governance Roles](./INTERFACES.md#imakinagovernable) → `IMakinaGovernable`
- [Registry Management](./INTERFACES.md#core-registry--context) → Registry interfaces
- [Factory Operations](./INTERFACES.md#factory-interfaces) → Factory interfaces

### For Integrators
- [Core Interfaces](./INTERFACES.md#makina-core-interfaces) → All core contracts
- [Periphery Interfaces](./INTERFACES.md#makina-periphery-interfaces) → Extended features
- [Oracle Integration](./INTERFACES.md#ioracleregistry) → `IOracleRegistry`
- [Flashloan Integration](./INTERFACES.md#iflashloonaggregator) → `IFlashloanAggregator`

## 📊 Documentation Statistics

| File | Lines | Size | Description |
|------|-------|------|-------------|
| INTERFACES.md | 873 | 32K | Comprehensive interface specs |
| ARCHITECTURE_DIAGRAM.md | 510 | 18K | Visual diagrams |
| INTERFACE_QUICK_REFERENCE.md | 280 | 9.2K | Quick lookup tables |
| README.md | 174 | 6.5K | Overview and guide |
| INDEX.md | - | - | This navigation file |

**Total:** ~1,837 lines of documentation

## 🔍 Search Guide

### By Interface Category

**Core Vaults:**
- [`IMachine`](./INTERFACES.md#imachine) - Main cross-chain vault
- [`ICaliber`](./INTERFACES.md#icaliber) - Strategy execution vault
- [`IPreDepositVault`](./INTERFACES.md#ipredepositvault) - Pre-launch vault

**Cross-Chain:**
- [`IBridgeController`](./INTERFACES.md#ibridgecontroller) - Bridge operations
- [`IBridgeAdapter`](./INTERFACES.md#ibridgeadapter) - Bridge protocol adapter
- [`ICaliberMailbox`](./INTERFACES.md#icalibermailbox) - Spoke endpoint

**Registries:**
- [`IHubCoreRegistry`](./INTERFACES.md#ihubcoreregistry) - Hub registry
- [`ISpokeCoreRegistry`](./INTERFACES.md#ispokecoreregistry) - Spoke registry
- [`IOracleRegistry`](./INTERFACES.md#ioracleregistry) - Oracle registry
- [`ITokenRegistry`](./INTERFACES.md#itokenregistry) - Token mapping

**Factories:**
- [`IHubCoreFactory`](./INTERFACES.md#ihubcorefactory) - Hub factory
- [`ISpokeCoreFactory`](./INTERFACES.md#ispokecorefactory) - Spoke factory
- [`IHubPeripheryFactory`](./INTERFACES.md#ihubperipheryfactory) - Periphery factory

**Periphery:**
- [`ISecurityModule`](./INTERFACES.md#isecuritymodule) - Protocol insurance
- [`IWatermarkFeeManager`](./INTERFACES.md#iwatermarkfeemanager) - Fee manager
- [`IAsyncRedeemer`](./INTERFACES.md#iasyncredeemer) - Async redemptions
- [`IFlashloanAggregator`](./INTERFACES.md#iflashloonaggregator) - Flashloans

### By Use Case

**Deposit/Withdraw:**
- [Deposit Flow Diagram](./ARCHITECTURE_DIAGRAM.md#data-flow-deposit-operation)
- [`IMachine.deposit()`](./INTERFACES.md#imachine)
- [`IDirectDepositor`](./INTERFACES.md#idirectdepositor)
- [`IAsyncRedeemer`](./INTERFACES.md#iasyncredeemer)

**Strategy Execution:**
- [Strategy Execution Diagram](./ARCHITECTURE_DIAGRAM.md#data-flow-strategy-execution)
- [`ICaliber.managePosition()`](./INTERFACES.md#icaliber)
- [`IWeirollVM`](./INTERFACES.md#iweirollvm)

**Cross-Chain Transfers:**
- [Bridge System Diagram](./ARCHITECTURE_DIAGRAM.md#bridge-system-architecture)
- [`IMachine.transferToSpokeCaliber()`](./INTERFACES.md#imachine)
- [`IBridgeController`](./INTERFACES.md#ibridgecontroller)

**Fee Management:**
- [`IFeeManager.calculateFixedFee()`](./INTERFACES.md#ifeemanager)
- [`IWatermarkFeeManager`](./INTERFACES.md#iwatermarkfeemanager)

**Security Operations:**
- [Security Module Lifecycle](./ARCHITECTURE_DIAGRAM.md#security-module-lifecycle)
- [`ISecurityModule.lock()`](./INTERFACES.md#isecuritymodule)
- [`ISecurityModule.slash()`](./INTERFACES.md#isecuritymodule)

## 🏗️ Architecture Diagrams

### High-Level Views
1. [Protocol Overview](./ARCHITECTURE_DIAGRAM.md#high-level-architecture)
2. [Hub Chain Architecture](./ARCHITECTURE_DIAGRAM.md#hub-chain-architecture-ethereum)
3. [Spoke Chain Architecture](./ARCHITECTURE_DIAGRAM.md#spoke-chain-architecture-l2--other-chains)

### Component Views
4. [Machine Component Diagram](./ARCHITECTURE_DIAGRAM.md#imachine-core-component-diagram)
5. [Caliber Component Diagram](./ARCHITECTURE_DIAGRAM.md#icaliber-core-component-diagram)
6. [Bridge System](./ARCHITECTURE_DIAGRAM.md#bridge-system-architecture)
7. [Periphery Components](./ARCHITECTURE_DIAGRAM.md#periphery-components-architecture)

### Flow Diagrams
8. [Deposit Flow](./ARCHITECTURE_DIAGRAM.md#data-flow-deposit-operation)
9. [Strategy Execution Flow](./ARCHITECTURE_DIAGRAM.md#data-flow-strategy-execution)
10. [Registry Lookup Flow](./ARCHITECTURE_DIAGRAM.md#registry-lookup-flow)
11. [Security Module Lifecycle](./ARCHITECTURE_DIAGRAM.md#security-module-lifecycle)

### Governance
12. [Governance Hierarchy](./ARCHITECTURE_DIAGRAM.md#governance--access-control-hierarchy)

## 📖 Reading Paths

### For New Developers
1. Start with [README.md](./README.md)
2. Review [Architecture Diagrams](./ARCHITECTURE_DIAGRAM.md)
3. Study core interfaces: `IMachine`, `ICaliber`
4. Review [Quick Reference](./INTERFACE_QUICK_REFERENCE.md)
5. Deep dive into [Full Interface Specs](./INTERFACES.md)

### For Integrators
1. [Interface Usage Guide](./README.md#-interface-usage-guide)
2. Relevant interface specs from [INTERFACES.md](./INTERFACES.md)
3. [Data Flow Diagrams](./ARCHITECTURE_DIAGRAM.md)

### For Auditors
1. [Full Interface Specs](./INTERFACES.md)
2. [Architecture Diagrams](./ARCHITECTURE_DIAGRAM.md)
3. [Interface Dependencies](./INTERFACE_QUICK_REFERENCE.md#interface-dependency-graph)
4. Source code in `makina-core/src/` and `makina-periphery/src/`

## 🔗 Related Resources

- **Makina Core Source:** `makina-core/src/`
- **Makina Periphery Source:** `makina-periphery/src/`
- **Tests:** `makina-core/test/` and `makina-periphery/test/`

## ℹ️ Document Information

- **Created:** September 30, 2025
- **Version:** 1.0
- **Total Interfaces Documented:** 40 (26 Core + 14 Periphery)
- **Solidity Version:** 0.8.28
- **License:** MIT

---

**Need help?** Start with the [README.md](./README.md) or search for your interface in [INTERFACES.md](./INTERFACES.md).
