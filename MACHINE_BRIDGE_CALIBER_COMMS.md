# Machine → Bridge → Caliber Communications Map

This document diagrams the primary communications between `Machine` (hub), Bridge layer (Controller/Adapters + external bridge), and `Caliber`/`CaliberMailbox` (spoke). It focuses on outbound transfers (hub→spoke), inbound claims (spoke→hub), and accounting sync.

## High-Level Diagram

```
┌───────────────────────────┐          ┌──────────────────────────────┐          ┌───────────────────────────┐
│           User            │          │            Machine            │          │        Bridge Layer       │
│        (Mechanic)         │          │           (Hub L1)            │          │ (Controller + Adapters)  │
└────────────┬──────────────┘          └─────────────┬────────────────┘          └──────────────┬───────────┘
             │                                         │                                      │
             │ 1) transferToSpokeCaliber(chain,token,amount,minOut)                          │
             ├─────────────────────────────────────────►                                      │
             │                                         │ 2) schedule outbound transfer        │
             │                                         ├─────────────────────────────────────►│
             │                                         │                                      │ 3) send via external bridge
             │                                         │                                      ├──────────────► (Across/Wormhole/...)
             │                                         │                                      │
             │                                         │                                      │ 4) deliver on Spoke
             │                                         │                                      │◄──────────────
             │                                         │                                      │
             │                                         │                                      ▼
             │                                         │                          ┌───────────────────────────┐
             │                                         │                          │      Caliber Mailbox      │
             │                                         │                          │        (Spoke L2)         │
             │                                         │                          └──────────────┬───────────┘
             │                                         │                                      │
             │                                         │ 5) authorizeInBridgeTransfer         │
             │                                         ◄───────────────────────────────────────┤
             │                                         │                                      │ 6) claimInBridgeTransfer
             │                                         ◄───────────────────────────────────────┤
             │                                         │                                      │
             │                                         │                                      ▼
             │                                         │                          ┌───────────────────────────┐
             │                                         │                          │         Caliber           │
             │                                         │                          │     (Strategy Vault)      │
             │                                         │                          └──────────────┬───────────┘
             │                                         │                                      │
             │                                         │ 7) manageTransfer/notifyIncomingTransfer
             │                                         ◄───────────────────────────────────────┤
             │                                         │                                      │
             │                                         │ 8) updateSpokeCaliberAccountingData(response, sigs)
             │                                         ◄───────────────────────────────────────────────────────
             │                                         │
             ▼                                         │
         Operations continue (managePosition, accounting, swaps) on Spoke Caliber
```

## Call/Message Breakdown

- 1) `Machine.transferToSpokeCaliber(bridgeId, chainId, token, amount, minOut)`
  - Initiator: `mechanic` on hub
  - Guards: recovery mode off, mechanic-only, token registry mapping for spoke, spoke adapter set, limits (loss bps, minOut)
  - Events: `TransferToCaliber`

- 2) Machine → BridgeAdapter: `scheduleOutBridgeTransfer(...)`
  - Records outbound intent; emits `OutBridgeTransferScheduled`

- 3) BridgeAdapter → External Bridge
  - Bridge-specific send (Across/Wormhole/Hyperlane/etc.)

- 4) External Bridge → Spoke Delivery
  - Deliver tokens/message to `CaliberMailbox` on the spoke chain

- 5) Hub side `authorizeInBridgeTransfer(bridgeId, transferHash)` (or spoke-side analog on mailbox)
  - Permissions: operator/mechanic depending on side and function

- 6) `claimInBridgeTransfer(bridgeId, outTransferId)`
  - Finalizes inbound transfer; funds arrive at `CaliberMailbox` and are routed to `Caliber`

- 7) `CaliberMailbox.manageTransfer(token, amount, data)` → `Caliber.notifyIncomingTransfer(token, amount)`
  - Caliber updates internal accounting context and available balances

- 8) Accounting Sync: `Machine.updateSpokeCaliberAccountingData(response, signatures)`
  - Data Source: CCQ (e.g., Wormhole Query) response + guardian signatures
  - Purpose: Hub-side AUM consolidation and bridge state reconciliation



## **Flow Explanation**

### **Phase 1: Transfer Initiation (Hub Side)**

**Step 1: User Initiates Transfer**
- A user (with Mechanic role) calls `transferToSpokeCaliber()` on the Machine contract
- Parameters: chainId (destination), token address, amount, minOut (minimum output expected)
- This is the entry point for moving funds from Hub (L1) to Spoke (L2)

**Step 2: Token Validation**
- Machine checks the Token Registry to ensure there's a valid mapping for the token on the destination spoke chain
- Validates that the token is supported for cross-chain transfers

**Step 3: Schedule Transfer**
- Machine calls Bridge Controller's `scheduleOutBridgeTransfer()`
- Validates multiple conditions:
  - Recovery mode is OFF
  - Mechanic role permissions
  - Maximum loss basis points (bps) limits
  - MinOut slippage protection
- Emits `OutBridgeTransferScheduled` event

### **Phase 2: Bridge Execution**

**Step 4: Route to Adapter**
- Bridge Controller routes the request to the appropriate Bridge Adapter
- Each adapter is specific to a bridge protocol (Across, Wormhole, Hyperlane, etc.)

**Step 5: Cross-Chain Message Send**
- Bridge Adapter invokes the external bridge protocol's send function
- Uses bridge-specific logic to package tokens and message data
- Initiates the actual cross-chain transaction

**Step 6: Deliver to Spoke**
- External bridge protocol delivers tokens and message to the CaliberMailbox on the spoke chain
- This happens asynchronously based on the bridge's finality requirements

### **Phase 3: Spoke Side Processing**

**Step 7: Authorization**
- `authorizeInBridgeTransfer()` is called (can be on hub or spoke depending on flow)
- Verifies the transfer hash and bridge ID
- Requires operator/mechanic permissions
- Marks the transfer as authorized for claiming

**Step 8: Claim Transfer**
- `claimInBridgeTransfer()` is called on the spoke side
- Finalizes the inbound transfer
- Tokens arrive at CaliberMailbox and are ready for routing

**Step 9: Transfer Management**
- CaliberMailbox calls `manageTransfer()` with token, amount, and metadata
- This routes the received funds to the Caliber vault

**Step 10: Notify Caliber**
- Caliber vault receives `notifyIncomingTransfer()` call
- Updates internal accounting to reflect the new funds
- Makes funds available for strategy operations (swaps, positions, etc.)

### **Phase 4: Accounting Reconciliation**

**Step 11: Query Spoke State**
- Oracle system (CCQ - Cross-Chain Query) queries the spoke chain's state
- Retrieves current accounting data from the Caliber vault and mailbox

**Step 12: Guardian Signatures**
- CCQ response is validated by Guardian nodes
- Multiple guardians sign the response to ensure data integrity
- Provides cryptographic proof of the spoke chain state

**Step 13: Update Hub Accounting**
- Machine contract receives `updateSpokeCaliberAccountingData()` with response and signatures
- Updates hub-side view of spoke balances and bridge states
- Consolidates AUM (Assets Under Management) across all spokes
- Reconciles any pending bridge transfers

---

## **Key Points:**

- **Asynchronous Flow**: Steps 5-6 happen off-chain via external bridges
- **Two-Phase Commit**: Authorization (step 7) + Claim (step 8) ensures safety
- **Accounting Loop**: Step 11-13 continuously syncs spoke state back to hub
- **Guard Rails**: Multiple validation points at Machine, Bridge Controller, and Mailbox
- **Role-Based**: Different steps require different permissions (Mechanic, Operator, Risk Manager)

For a visual deep-dive, see: [Detailed flow view](https://claude.ai/public/artifacts/dc92cdd9-39d4-43d4-9d00-cf12cbd15c23)

## Error/Guardrails (where enforced)

- Machine (Hub):
  - Recovery mode pause on transfer ops
  - Mechanic-only for transfer scheduling
  - Token registry check for foreign token mapping
  - Spoke adapter required per chain/bridge
  - Max loss bps and `minOut` validation
  - Pending bridge transfer protection, bridge state mismatch checks

- Caliber Mailbox (Spoke):
  - Operator/mechanic role checks for send/authorize/claim paths
  - Per-bridge enable/disable and loss bps limits
  - Reset functions for abnormal states

- Bridge Controller/Adapters:
  - Adapter existence enforced
  - Out-transfer enable/disable and max loss bps managed by risk manager/timelock

## Reference Tests (for behavior validation)

- Machine transfers: `test/integration/concrete/machine/transfer-to-spoke-caliber/transferToSpokeCaliber.t.sol`
- Bridge controller toggles/queries: `test/integration/concrete/bridge-controller/*`
- Mailbox send/authorize/claim/cancel: `test/integration/concrete/caliber-mailbox/*`
- Accounting sync (hub-side): exercised via `updateSpokeCaliberAccountingData` usages in Machine tests

## Notes

- External bridge delivery is abstracted by adapters; protocol enforces local safety and consistency.
- Hub AUM relies on signed CCQ responses to reconcile spoke accounting and bridge states.
