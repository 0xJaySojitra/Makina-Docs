# Makina Protocol Architecture Diagram

This document provides visual diagrams showing how all interfaces relate to each other in the Makina protocol.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MAKINA PROTOCOL                                    │
│                                                                              │
│  ┌───────────────────────────┐        ┌───────────────────────────┐        │
│  │     MAKINA CORE           │        │   MAKINA PERIPHERY        │        │
│  │   (Cross-Chain Vaults)    │◄──────►│  (Extended Features)      │        │
│  └───────────────────────────┘        └───────────────────────────┘        │
│              │                                    │                          │
│              │                                    │                          │
│  ┌───────────▼────────────┐          ┌───────────▼────────────┐            │
│  │   Hub Chain (Ethereum) │          │ Spoke Chains (L2s, etc)│            │
│  └────────────────────────┘          └────────────────────────┘            │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Hub Chain Architecture (Ethereum)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              HUB CHAIN                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    IHubCoreRegistry                              │      │
│  │  ┌────────────┬────────────┬────────────┬────────────────────┐  │      │
│  │  │ Core       │ Oracle     │ Token      │ Chain              │  │      │
│  │  │ Factory    │ Registry   │ Registry   │ Registry           │  │      │
│  │  └────────────┴────────────┴────────────┴────────────────────┘  │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                   │                                          │
│                          ┌────────┴────────┐                                │
│                          ▼                 ▼                                 │
│         ┌────────────────────────┐  ┌─────────────────────┐                │
│         │  IHubCoreFactory       │  │ IHubPeripheryFactory │               │
│         └───────┬────────────────┘  └──────┬──────────────┘                │
│                 │                           │                                │
│     ┌───────────┼───────────┐               │                               │
│     ▼           ▼           ▼               ▼                               │
│  ┌─────┐  ┌─────────┐  ┌──────┐    ┌────────────────┐                     │
│  │Pre- │  │Machine  │  │Bridge│    │  Depositor/    │                     │
│  │Dep  │─►│(IMachine)│  │Adapt.│    │  Redeemer/     │                     │
│  │Vault│  └────┬────┘  └──────┘    │  FeeManager/   │                     │
│  └─────┘       │                     │  SecurityMod   │                     │
│                │                     └────────────────┘                     │
│                ▼                                                             │
│         ┌──────────────┐                                                    │
│         │ Hub Caliber  │                                                    │
│         │  (ICaliber)  │                                                    │
│         └──────────────┘                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Spoke Chain Architecture (L2 / Other Chains)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SPOKE CHAIN                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                  ISpokeCoreRegistry                              │      │
│  │  ┌────────────┬────────────┬────────────┬────────────────────┐  │      │
│  │  │ Core       │ Oracle     │ Token      │ Caliber Mailbox    │  │      │
│  │  │ Factory    │ Registry   │ Registry   │ Beacon             │  │      │
│  │  └────────────┴────────────┴────────────┴────────────────────┘  │      │
│  └──────────────────────────────────────────────────────────────────┘      │
│                                   │                                          │
│                                   ▼                                          │
│                      ┌────────────────────────┐                             │
│                      │  ISpokeCoreFactory     │                             │
│                      └──────┬─────────────────┘                             │
│                             │                                                │
│                   ┌─────────┴──────────┐                                    │
│                   ▼                    ▼                                     │
│           ┌───────────────┐    ┌──────────────┐                            │
│           │Spoke Caliber  │◄───│Caliber       │                            │
│           │  (ICaliber)   │    │Mailbox       │                            │
│           └───────────────┘    │(IMbox)       │                            │
│                                └──────────────┘                             │
│                                        │                                     │
│                                        ▼                                     │
│                                ┌───────────────┐                            │
│                                │ Bridge        │                            │
│                                │ Adapters      │                            │
│                                └───────────────┘                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## IMachine Core Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            IMachine                                          │
│                     (Cross-Chain Vault Manager)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Inherits: IMachineEndpoint, IMakinaGovernable                              │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                    Components                                    │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  • IMachineShare (shareToken)                                   │       │
│  │  • IFeeManager (feeManager)                                     │       │
│  │  • IDepositor (depositor)                                       │       │
│  │  • IRedeemer (redeemer)                                         │       │
│  │  • ICaliber (hubCaliber)                                        │       │
│  │  • ISecurityModule (via feeManager)                             │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                   Spoke Calibers                                 │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  ChainId 1 → ICaliberMailbox → ICaliber                        │       │
│  │  ChainId 2 → ICaliberMailbox → ICaliber                        │       │
│  │  ChainId N → ICaliberMailbox → ICaliber                        │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                  Bridge Adapters                                 │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  Bridge 1 (e.g., Across)   → IBridgeAdapter                    │       │
│  │  Bridge 2 (e.g., Wormhole) → IBridgeAdapter                    │       │
│  │  Bridge N                  → IBridgeAdapter                    │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## ICaliber Core Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            ICaliber                                          │
│                      (Strategy Execution Vault)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                    Components                                    │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  • IWeirollVM (weirollVm)                                       │       │
│  │  • IMachineEndpoint (hubMachineEndpoint)                        │       │
│  │  • ISwapModule (via context)                                    │       │
│  │  • IOracleRegistry (via context)                                │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                      Positions                                   │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  Position 1: {id, value, isDebt, lastAccountingTime}           │       │
│  │  Position 2: {id, value, isDebt, lastAccountingTime}           │       │
│  │  Position N: {id, value, isDebt, lastAccountingTime}           │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                    Base Tokens                                   │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  • USDC, USDT, DAI, etc.                                        │       │
│  │  • Accounting Token (reference)                                 │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │              Instruction Management                              │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │  allowedInstrRoot (Merkle Root)                                 │       │
│  │  ├─ MANAGEMENT Instructions                                     │       │
│  │  ├─ ACCOUNTING Instructions                                     │       │
│  │  ├─ HARVEST Instructions                                        │       │
│  │  └─ FLASHLOAN_MANAGEMENT Instructions                           │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Bridge System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BRIDGE SYSTEM                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Hub Machine                                     Spoke Caliber Mailbox      │
│  ┌──────────────────────┐                      ┌──────────────────────┐    │
│  │  IMachine            │                      │  ICaliberMailbox     │    │
│  │    (Chain A)         │                      │    (Chain B)         │    │
│  └──────┬───────────────┘                      └──────┬───────────────┘    │
│         │ transferToSpokeCaliber                      │ manageTransfer      │
│         ▼                                             ▼                      │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │             IBridgeController                                    │       │
│  └──────┬──────────────────────────────────────────────────────────┘       │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │             IBridgeAdapter (per Bridge ID)                       │       │
│  ├─────────────────────────────────────────────────────────────────┤       │
│  │                                                                  │       │
│  │  scheduleOutBridgeTransfer() ──┐                                │       │
│  │  sendOutBridgeTransfer()       │                                │       │
│  │  authorizeInBridgeTransfer()   │                                │       │
│  │  claimInBridgeTransfer()       │                                │       │
│  │  cancelOutBridgeTransfer()     │                                │       │
│  └────────────────────────────────┼────────────────────────────────┘       │
│                                   │                                          │
│                                   ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │              External Bridge Protocol                            │       │
│  │  (Across, Wormhole, Hyperlane, etc.)                            │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Periphery Components Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PERIPHERY COMPONENTS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                           IHubPeripheryRegistry                              │
│                                   │                                          │
│                                   ▼                                          │
│                       IHubPeripheryFactory                                   │
│                                   │                                          │
│           ┌───────────────────────┼────────────────────────┐                │
│           │                       │                        │                 │
│           ▼                       ▼                        ▼                 │
│  ┌────────────────┐    ┌──────────────────┐   ┌──────────────────┐        │
│  │  Depositors    │    │   Redeemers      │   │  Fee Managers    │        │
│  ├────────────────┤    ├──────────────────┤   ├──────────────────┤        │
│  │                │    │                  │   │                  │        │
│  │ IDirectDep.    │    │ IAsyncRedeemer   │   │ IWatermarkFee    │        │
│  │ (Impl ID: 0)   │    │ (Impl ID: 0)     │   │ Manager          │        │
│  │                │    │                  │   │ (Impl ID: 0)     │        │
│  │ Future Impls   │    │ Future Impls     │   │ Future Impls     │        │
│  │ (Impl ID: 1+)  │    │ (Impl ID: 1+)    │   │ (Impl ID: 1+)    │        │
│  │                │    │                  │   │                  │        │
│  └────────┬───────┘    └────────┬─────────┘   └────────┬─────────┘        │
│           │                     │                      │                    │
│           └─────────────────────┼──────────────────────┘                    │
│                                 │                                            │
│                                 ▼                                            │
│                          ┌─────────────┐                                    │
│                          │  IMachine   │                                    │
│                          └─────────────┘                                    │
│                                 │                                            │
│                                 │ Fee Distribution                           │
│                                 ▼                                            │
│                       ┌──────────────────┐                                  │
│                       │ ISecurityModule  │                                  │
│                       ├──────────────────┤                                  │
│                       │ • Lock Shares    │                                  │
│                       │ • Start Cooldown │                                  │
│                       │ • Redeem         │                                  │
│                       │ • Slash          │                                  │
│                       └──────────────────┘                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Governance & Access Control Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    GOVERNANCE HIERARCHY                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         IMakinaGovernable                                    │
│                                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│  │   Authority      │  │  Security        │  │  Risk            │         │
│  │   (Owner)        │  │  Council         │  │  Manager         │         │
│  │                  │  │                  │  │                  │         │
│  │ • Set all roles  │  │ • Recovery mode  │  │ • Update params  │         │
│  │ • Ultimate ctrl  │  │ • Emergency stop │  │ • Set limits     │         │
│  └──────────────────┘  └──────────────────┘  └────────┬─────────┘         │
│                                                        │                     │
│                                              ┌─────────▼─────────┐          │
│                                              │  Risk Manager     │          │
│  ┌──────────────────┐  ┌──────────────────┐ │  Timelock         │          │
│  │   Mechanic       │  │  Operators       │ │                   │          │
│  │                  │  │                  │ │ • Delayed actions │          │
│  │ • Execute strats │  │ • Routine ops    │ │ • Param updates   │          │
│  │ • Manage posits  │  │ • Bridge ops     │ └───────────────────┘          │
│  │ • Update AUM     │  │ • Accounting     │                                 │
│  └──────────────────┘  └──────────────────┘                                 │
│                                                                              │
│  Recovers to →  Recovery Mode (all operations paused)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow: Deposit Operation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DEPOSIT FLOW                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  User                                                                        │
│   │                                                                          │
│   │  1. Approve accounting tokens                                           │
│   ├──────────────────────────►  ERC20 (Accounting Token)                   │
│   │                                                                          │
│   │  2. deposit(assets, receiver, minShares)                                │
│   ├──────────────────────────► IDirectDepositor                             │
│   │                                  │                                       │
│   │                                  │ 3. deposit()                          │
│   │                                  ├────────────► IMachine                 │
│   │                                  │                  │                    │
│   │                                  │                  │ 4. convertToShares │
│   │                                  │                  │    (uses totalAUM) │
│   │                                  │                  │                    │
│   │                                  │                  │ 5. Transfer tokens │
│   │                                  │                  ├──────────►         │
│   │                                  │                  │         Accounting │
│   │                                  │                  │         Token      │
│   │                                  │                  │                    │
│   │                                  │                  │ 6. Mint shares     │
│   │                                  │                  ├──────────►         │
│   │                                  │                  │      IMachineShare │
│   │                                  │                  │                    │
│   │                                  │    7. Return     │                    │
│   │                                  │◄─────────────────┤                    │
│   │     8. Receive shares            │                                       │
│   ◄──────────────────────────────────┤                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Data Flow: Strategy Execution

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STRATEGY EXECUTION FLOW                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Mechanic                                                                    │
│   │                                                                          │
│   │  1. managePosition(mgmtInstr, acctInstr)                                │
│   ├──────────────────────────► ICaliber                                     │
│   │                                  │                                       │
│   │                                  │ 2. Verify Merkle proof                │
│   │                                  │    (against allowedInstrRoot)         │
│   │                                  │                                       │
│   │                                  │ 3. Execute management instruction     │
│   │                                  ├────────────► IWeirollVM               │
│   │                                  │                  │                    │
│   │                                  │                  │ 4. Execute cmds    │
│   │                                  │                  │    - Approve       │
│   │                                  │                  │    - Call external │
│   │                                  │                  │    - Transfer      │
│   │                                  │                  │                    │
│   │                                  │    5. Return     │                    │
│   │                                  │◄─────────────────┤                    │
│   │                                  │                                       │
│   │                                  │ 6. Execute accounting instruction     │
│   │                                  ├────────────► IWeirollVM               │
│   │                                  │                  │                    │
│   │                                  │                  │ 7. Query position  │
│   │                                  │                  │    value           │
│   │                                  │                  │                    │
│   │                                  │    8. Return     │                    │
│   │                                  │◄─────────────────┤                    │
│   │                                  │                                       │
│   │                                  │ 9. Validate value change              │
│   │                                  │    (check loss limits)                │
│   │                                  │                                       │
│   │                                  │ 10. Update position state             │
│   │                                  │     - Update value                    │
│   │                                  │     - Update lastAccountingTime       │
│   │                                  │                                       │
│   │     11. Return (value, change)   │                                       │
│   ◄──────────────────────────────────┤                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Registry Lookup Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      REGISTRY LOOKUP FLOW                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Contract (e.g., Caliber)                                                    │
│   │                                                                          │
│   │  Need: Swap Module                                                      │
│   ├──────────────────────────► IMakinaContext                               │
│   │                                  │                                       │
│   │                                  │ registry()                            │
│   │                                  ├────────────► ICoreRegistry            │
│   │                                  │                  │                    │
│   │                                  │                  │ swapModule()       │
│   │                                  │                  ├──────────►         │
│   │                                  │                  │      ISwapModule   │
│   │                                  │    Return addr   │                    │
│   │                                  │◄─────────────────┤                    │
│   │     Return ISwapModule addr      │                                       │
│   ◄──────────────────────────────────┤                                       │
│   │                                                                          │
│   │  Now can call: swap(order)                                              │
│   ├───────────────────────────────────────────────────► ISwapModule         │
│                                                                              │
│  Similar flow for:                                                           │
│  • oracleRegistry() → IOracleRegistry                                       │
│  • tokenRegistry() → ITokenRegistry                                         │
│  • flashLoanModule() → Flashloan contracts                                  │
│  • etc.                                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Security Module Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  SECURITY MODULE LIFECYCLE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. LOCK PHASE                                                               │
│     User → lock(machineShares) → ISecurityModule                            │
│                                   │                                          │
│                                   ├─ Transfer machineShares from user       │
│                                   ├─ Mint security shares to user           │
│                                   └─ Increase totalLockedAmount             │
│                                                                              │
│  2. HOLDING PHASE                                                            │
│     • Security shares earn fees from protocol                               │
│     • convertToAssets() ratio may increase over time                        │
│                                                                              │
│  3. UNLOCK PHASE (Start Cooldown)                                           │
│     User → startCooldown(shares) → ISecurityModule                          │
│                                   │                                          │
│                                   ├─ Burn security shares                   │
│                                   ├─ Mint cooldown NFT                      │
│                                   │   (ISMCooldownReceipt)                  │
│                                   ├─ Record: shares, maxAssets, maturity    │
│                                   └─ maturity = now + cooldownDuration      │
│                                                                              │
│  4. WAITING PHASE                                                            │
│     • Cooldown period (e.g., 7 days)                                        │
│     • User holds NFT receipt                                                │
│     • Can cancel via cancelCooldown()                                       │
│                                                                              │
│  5. REDEMPTION PHASE                                                         │
│     User → redeem(cooldownId) → ISecurityModule                             │
│                                   │                                          │
│                                   ├─ Check maturity passed                  │
│                                   ├─ Calculate assets (may be less if slash)│
│                                   ├─ Burn cooldown NFT                      │
│                                   ├─ Transfer machineShares to user         │
│                                   └─ Decrease totalLockedAmount             │
│                                                                              │
│  SLASHING (can happen anytime)                                              │
│     Governor → slash(amount) → ISecurityModule                              │
│                                   │                                          │
│                                   ├─ Reduce totalLockedAmount               │
│                                   ├─ Enter slashing mode                    │
│                                   ├─ Pause normal operations                │
│                                   └─ Await settleSlashing()                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

**Document Version:** 1.0  
**Last Updated:** September 30, 2025  
**Related:** [INTERFACES.md](./INTERFACES.md), [INTERFACE_QUICK_REFERENCE.md](./INTERFACE_QUICK_REFERENCE.md)
