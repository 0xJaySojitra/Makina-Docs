# Makina Core – Detailed Test Walkthroughs (Contract-Level Flows)

Purpose: Help you understand the protocol at contract level by walking through the most important tests that demonstrate end-to-end behavior, invariants, and failure paths. Each section explains why the test exists, what it proves, and what to pay attention to for audits.

## 1) Machine Initialization (with/without Pre-Deposit)

- Path: `makina-core/test/integration/concrete/machine/initialize/initialize.t.sol`
- Why it matters:
  - Ensures a Machine can only initialize with a priceable accounting token (prevents unpriced AUM).
  - Validates share token ownership transfer and correct link to hub caliber and registry.
  - Covers migration from `PreDepositVault` (common deployment flow).
- Core flows and checks:
  - Revert when accounting token has no oracle route (`PriceFeedRouteNotRegistered`).
  - Revert if share token ownership not transferred to Machine.
  - State assertions after init: roles (mechanic, securityCouncil, depositor, redeemer), token addresses, thresholds, authority, idle token.
  - With pre-deposit: deploy caliber via beacon, migrate pre-deposit, assert `migrated()` and post-state.
- Audit takeaways:
  - Initialization guards are strict; ownership and priceability are enforced.
  - Ensure deployment tooling mirrors these expectations (factory + beacons + authority wiring).

## 2) Deposit – Minting Shares Against AUM

- Path: `makina-core/test/integration/concrete/machine/deposit/deposit.t.sol`
- Why it matters:
  - Validates ERC4626-like deposit semantics with protocol-specific guards.
  - Exercises slippage protection and share limits at entry.
  - Asserts AUM progression on deposit.
- Core flows and checks:
  - Reentrancy guard: forced ERC20 callback reenters `deposit` → revert.
  - Recovery mode block: deposit reverts when paused.
  - Role guard: only `depositor` may call.
  - Limits: `maxMint` via `shareLimit` enforced from zero supply case.
  - Slippage: `minSharesOut` enforced.
  - Happy path: balances, `Deposit` event, `lastTotalAum` equals input.
- Audit takeaways:
  - Entry is well-guarded. Focus review on `convertToShares` math and oracle/AUM invariants.

## 3) Redeem – Burning Shares for Assets

- Path: `makina-core/test/integration/concrete/machine/redeem/redeem.t.sol`
- Why it matters:
  - Confirms redemption respects AUM, slippage, and availability constraints.
  - Validates reentrancy and role controls at exit.
- Core flows and checks:
  - Reentrancy guard: forced callback reenters `redeem` → revert.
  - Recovery mode block.
  - Role guard: only `redeemer` may call.
  - `maxWithdraw` enforcement when assets moved to caliber.
  - Slippage: `minAssetsOut` enforced.
  - Partial and full redeem: balance deltas, events, AUM tracking to zero.
- Audit takeaways:
  - Exit path protection is solid. Review `convertToAssets` math + availability when assets reside in caliber.

## 4) Caliber – Managing Positions via Weiroll (Instruction Root + Proofs)

- Path: `makina-core/test/integration/concrete/caliber/manage-position/managePosition.t.sol`
- Why it matters:
  - This is the core of strategy execution: Weiroll programs must be proven against a merkle root.
  - Many failure cases ensure proofs and instruction pairing are correct.
- Core flows and checks:
  - Reentrancy: reentering `managePosition` via token callback → revert.
  - Role guard: only `mechanic` may call.
  - Input validation: `positionId != 0`, mgmt vs accounting pairing, isDebt mismatch, groupId mismatch.
  - Merkle proof rejections: wrong vault, positionId, isDebt, groupId, affectedTokens, commands, state, bitmap.
  - Timelocked `allowedInstrRoot` updates: allowed while pending, disallowed after effective wrong root, allowed after correct root.
- Audit takeaways:
  - The proof surface is large; tests cover many dimensions. Review helper builders to understand instruction encoding.

## 5) Caliber – Detailed AUM and Accounting Freshness

- Path: `makina-core/test/integration/concrete/caliber/get-detailed-aum/getDetailedAum.t.sol`
- Why it matters:
  - Shows how AUM is derived from base tokens and positions, including debt handling and staleness.
- Core flows and checks:
  - Staleness: `PositionAccountingStale` after threshold.
  - Zero AUM case and unregistered tokens behavior.
  - Accounting token vs base token composition.
  - Non-debt vs debt positions reflected in net AUM and encoded outputs.
  - Batch accounting effects with rate changes.
- Audit takeaways:
  - Accounting freshness is enforced; debt updates impact net AUM as expected. Review encoding/decoding and aggregation math.

## 6) Bridging – Transfer to Spoke Caliber (Outbound Scheduling)

- Path: `makina-core/test/integration/concrete/machine/transfer-to-spoke-caliber/transferToSpokeCaliber.t.sol`
- Why it matters:
  - Validates hub → spoke transfer scheduling, adapter wiring, loss limits, and bridge state integrity.
- Core flows and checks:
  - Reentrancy guard; recovery mode; role guard (mechanic-only).
  - Token registry checks for foreign tokens; valid chainId; per-spoke adapter configured.
  - Guardrails: pending transfers, bridge state mismatch with CCQ responses, out-transfer disabling, `MaxValueLossExceeded`, `MinOutputAmountExceedsInputAmount`.
  - Happy paths: scheduling events, idle token invariants, partial vs full balance transfers.
- Audit takeaways:
  - Protocol-side safety is asserted; external bridge reliability is out-of-scope. Review message hashing and state sync logic.

## 7) Bridge Controller – Adapter Creation and Queries

- Paths:
  - `makina-core/test/integration/concrete/bridge-controller/get-bridge-adapter/getBridgeAdapter.t.sol`
  - `makina-core/test/integration/concrete/bridge-controller/create-bridge-adapter/createBridgeAdapter.t.sol`
- Why it matters:
  - Ensures adapters exist before usage and can be created only via governed roles.
- Core flows and checks:
  - Missing adapter reverts.
  - Creation emits, returns consistent addresses.
  - Risk/timelock knobs tested elsewhere in dir: enable/disable, loss bps.
- Audit takeaways:
  - Verify controller’s storage mappings and role gating; ensure per-bridge invariants.

## 8) Factory – Full Machine Deployment

- Path: `makina-core/test/integration/concrete/hub-core-factory/create-machine/createMachine.t.sol`
- Why it matters:
  - Full deployment path: Machine + Caliber + Share token via beacons and create salts.
- Core flows and checks:
  - Access-managed caller; zero/used salts reverts; CREATE3 proxy/contract failure cases.
  - Emits `CaliberCreated` and `MachineCreated`.
  - Post-state assertions: registry, roles, thresholds, authority; `isIdleToken`; spoke calibers length; share token metadata and minter.
- Audit takeaways:
  - Deployment is robust. Pay attention to salt derivation, beacon addresses, and ownership transfers.

## 9) Machine – Limits and AUM Updates

- Paths:
  - `makina-core/test/integration/concrete/machine/max-mint/*`
  - `makina-core/test/integration/concrete/machine/max-withdraw/*`
  - `makina-core/test/integration/concrete/machine/update-total-aum/*`
- Why it matters:
  - Documents guardrails on mint/withdraw capacity and global AUM synchronization.
- Core flows and checks:
  - `maxMint` == `shareLimit` at zero supply; withdraw ceiling when assets in caliber.
  - AUM recomputation and timestamp progression.
- Audit takeaways:
  - Limits behave consistently with supply and availability; AUM cadence is explicit.

## How to Use These Walkthroughs

- Read sections 1→6 in order for the fastest grasp of core flows (init, deposit, AUM, managePosition, bridging, redeem).
- Keep the test file open alongside contract code to map asserts to state changes.
- When you see helpers (mocks/builders), open `test/base` and `test/utils` to understand encodings.

## Gaps and Suggested Deep-Dives

- Fuzzing: Explore `test/*/fuzz` directories for edge distributions on math and state transitions.
- Role/timelock matrices: Cross-check registry/factory unit tests for full setter coverage.
- Cross-chain timing: Consider skew/race scenarios around CCQ and accounting timestamps.
- External integration behavior: Out-of-scope for tests (bridges/oracles), but important to mentally model.
