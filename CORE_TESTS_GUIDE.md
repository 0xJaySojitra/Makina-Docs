# Makina Core – Test Files Guide (Contract-Level Flows)

This guide lists the most useful `makina-core` test files to understand protocol behavior by area. Each entry states the flow covered and what to look for. Use this as a starting map to read tests in order.

## Machine (Vault) – Integration Tests

- `makina-core/test/integration/concrete/machine/initialize/initialize.t.sol`
  - **Flows**: Machine init with/without pre-deposit; share token ownership; registry hooks; hub caliber linkage.
  - **Look for**: Reverts on non-priceable accounting token; ownership transfer; state after init.

- `makina-core/test/integration/concrete/machine/deposit/deposit.t.sol`
  - **Flows**: Direct deposit; maxMint/shareLimit; slippage guard; recovery mode; reentrancy guard.
  - **Look for**: `Deposit` event; balances after deposit; `lastTotalAum` update.

- `makina-core/test/integration/concrete/machine/redeem/redeem.t.sol`
  - **Flows**: Partial and full redeem; maxWithdraw; slippage guard; recovery mode; reentrancy guard.
  - **Look for**: `Redeem` event; balance deltas; AUM after redemption.

- `makina-core/test/integration/concrete/machine/update-total-aum/*`
  - **Flows**: Global AUM sync/update.
  - **Look for**: AUM recomputation and timestamps.

- Bridging (hub → spoke):
  - `makina-core/test/integration/concrete/machine/transfer-to-spoke-caliber/transferToSpokeCaliber.t.sol`
    - **Flows**: Scheduling outbound bridge transfers; checks on chainId, adapters, limits, loss bps.
    - **Look for**: Outbound scheduling events; idle token state; guardrails (pending transfer, mismatch, max loss, etc.).
  - `makina-core/test/integration/concrete/machine/send-out-bridge-transfer/*`
    - **Flows**: Sending the scheduled outbound transfer.
  - `makina-core/test/integration/concrete/machine/authorize-in-bridge-transfer/*` and `claim-in-bridge-transfer/*`
    - **Flows**: Inbound authorization/claim on hub side; operator permissions.
  - `makina-core/test/integration/concrete/machine/reset-bridging-state/*`
    - **Flows**: Reset bridge state by security council.

- Views & Limits:
  - `makina-core/test/integration/concrete/machine/max-mint/*`, `max-withdraw/*`
    - **Flows**: Capacity checks for mint/withdraw limits.

## Caliber (Strategy Engine) – Integration Tests

- `makina-core/test/integration/concrete/caliber/manage-position/managePosition.t.sol`
  - **Flows**: Management + accounting instruction execution via Weiroll; merkle proof checks; role checks.
  - **Look for**: Instruction mismatch cases; invalid proofs; timelocked root updates behavior.

- `makina-core/test/integration/concrete/caliber/get-detailed-aum/getDetailedAum.t.sol`
  - **Flows**: Detailed AUM computation; base tokens vs positions; stale accounting; debt vs non-debt.
  - **Look for**: Net AUM composition; stale thresholds; batch accounting updates.

- Other key flows (directories):
  - `account-for-position/*`, `account-for-position-batch/*`
  - `swap/*`, `harvest/*`, `manage-flashloan/*`, `notify-incoming-transfer/*`, `transfer-to-hub-machine/*`
  - **Look for**: Operator-only paths, loss bps checks, module interactions.

## Bridge Controller – Integration Tests

- `makina-core/test/integration/concrete/bridge-controller/get-bridge-adapter/getBridgeAdapter.t.sol`
  - **Flows**: Adapter lookup and missing adapter revert.

- `makina-core/test/integration/concrete/bridge-controller/create-bridge-adapter/createBridgeAdapter.t.sol`
  - **Flows**: Creating adapters; role restrictions; wiring adapter into controller.

- `makina-core/test/integration/concrete/bridge-controller/*`
  - `is-bridge-supported/*`, `is-out-transfer-enabled/*`, `get-max-bridge-loss-bps/*`, `set-*/*`
  - **Look for**: Risk manager timelock controls; enable/disable paths.

## Factories – Integration Tests

- `makina-core/test/integration/concrete/hub-core-factory/create-machine/createMachine.t.sol`
  - **Flows**: Full machine deployment (machine + caliber + share token); salts; emits; post-invariants.
  - **Look for**: Access-managed deploys; expected registries/roles; beacons and ownership.

- `makina-core/test/integration/concrete/hub-core-factory/create-pre-deposit-vault/*`, `create-machine-from-pre-deposit/*`
  - **Flows**: Pre-deposit flow and migration into machine.

- `makina-core/test/integration/concrete/spoke-core-factory/*`
  - **Flows**: Spoke-side deployments: caliber, mailbox, adapters.

## Registries – Unit/Integration

- `makina-core/test/unit/concrete/core-registry/CoreRegistry.t.sol`
  - **Flows**: Registry getters/setters; component wiring and events.

- `makina-core/test/unit/concrete/hub-core-registry/*`, `spoke-core-registry/*`, `chain-registry/*`, `oracle-registry/*`, `token-registry/*`
  - **Look for**: Proper access control and validation in config paths.

## Mailbox (Spoke) – Integration Tests

- `makina-core/test/integration/concrete/caliber-mailbox/*`
  - **Flows**: manageTransfer, send/authorize/claim/cancel in-bridge/out-bridge transfer paths; spoke accounting.
  - **Look for**: Operator/mechanic permissions; per-bridge config; reset functions.

## Base and Utilities

- `makina-core/test/base/Base.t.sol`
  - **Purpose**: Common setup, roles, mocks, helpers shared across tests.

- `makina-core/test/utils/*`, `test/mocks/*`
  - **Purpose**: Helpers for wormhole, tokens, 4626, modules; good to skim when a test references a helper.

---

Suggested reading order for understanding end-to-end flow:
1) Machine initialize → deposit → redeem → update-total-aum
2) Caliber managePosition → getDetailedAum → account-for-position/batch
3) Bridging with Machine transfer-to-spoke → send-out/authorize/claim → Mailbox
4) Factory create-machine (+ pre-deposit variants)
5) Registries and Bridge Controller controls
