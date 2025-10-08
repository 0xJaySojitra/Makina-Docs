# Audit – Utility Libraries (DecimalsUtils, MachineUtils)

Scope: Review of `makina-core/src/libraries/DecimalsUtils.sol` and `makina-core/src/libraries/MachineUtils.sol` focusing on correctness, edge cases, and security implications in current protocol flows.

---

## DecimalsUtils.sol

Path: `makina-core/src/libraries/DecimalsUtils.sol`

### What it does
- Provides token decimals discovery via `_getDecimals(address asset)` using a `staticcall` to `IERC20Metadata.decimals()`.
- Defines constants for defaults and share token units:
  - `DEFAULT_DECIMALS = 18`
  - `MIN_DECIMALS = 6` (not used here)
  - `MAX_DECIMALS = 18`
  - `SHARE_TOKEN_DECIMALS = 18`
  - `SHARE_TOKEN_UNIT = 10 ** 18`

### Key behavior
- `_getDecimals` decodes the return data as `uint256` and only accepts it if `<= type(uint8).max`, then casts to `uint8`.
- If the call fails or the value is invalid/too large, defaults to 18.

### Edge cases and observations
- Non-standard tokens:
  - Tokens with missing `decimals()` or returning malformed data fall back to 18.
  - If a token actually has fewer/more decimals but the call fails, accounting using this value may be skewed.
- Return type handling:
  - Standard OpenZeppelin `IERC20Metadata.decimals()` returns `uint8`. Decoding as `uint256` is fine given the later cap check, but it’s slightly non-idiomatic.
- Constants usage:
  - `MIN_DECIMALS` and `MAX_DECIMALS` are declared but unused for validation (only `uint8` max is enforced). This is acceptable; just note they don’t constrain `_getDecimals`.

### Potential risks
- Fallback-to-18 behavior can cause mispricing if:
  - A token wrongly fails `decimals()` and has non-18 decimals.
  - This impacts `_accountingValueOf` (see MachineUtils) when scaling by `10 ** decimals(token)`.
- Not critical if the Token Registry filters supported tokens and the Oracle Registry normalizes pricing consistently.

### Recommendations
- Optional: add an allowlist in Token Registry with stored decimals for supported tokens to avoid relying on live `decimals()` for critical tokens.
- Optional: surface metrics/telemetry when defaulting to 18 to ease detection during ops.

---

## MachineUtils.sol

Path: `makina-core/src/libraries/MachineUtils.sol`

### What it does (high level)
- Core accounting helpers used by `Machine`:
  - `updateTotalAum`: recomputes and stamps global AUM and last accounting time
  - `manageFees`: mints, approves, distributes, and cleans up fee shares (fixed + performance) with accrual caps
  - `updateSpokeCaliberAccountingData`: verifies CCQ responses and updates per-chain spoke data
  - `migrateFromPreDeposit`: migrates from `PreDepositVault` and initializes state
  - `getSharePrice`: share price formula with decimals offset handling
  - Private helpers for CCQ handling, bridge state reconciliation, and accounting value conversion

### Detailed behaviors

- `updateTotalAum($, oracleRegistry)`
  - Sets `$_lastTotalAum` from `_getTotalAum` and updates `$_lastGlobalAccountingTime`.
  - Returns new AUM.

- `manageFees($)`
  - Computes `elapsedTime` since last mint.
  - If `elapsedTime >= feeMintCooldown`:
    - Reads `currentShareSupply` from `IMachineShare.totalSupply()`.
    - Fixed fee: `min(FeeManager.calculateFixedFee(supply, elapsed), (supply * elapsed) * maxFixedRate / 1e18)`.
    - Computes `netSharePrice` with a supply including `fixedFee` to offset perf fee base.
    - Performance fee: `min(FeeManager.calculatePerformanceFee(supply, lastFeeSharePrice, netSharePrice, elapsed), (supply * elapsed) * maxPerfRate / 1e18)`.
    - Mints total fee shares to Machine, approves FeeManager, calls `distributeFees(fixed, perf)`, resets approval.
    - Burns any dust shares left on Machine to avoid residual supply drift.
    - Updates `$_lastMintedFeesTime` and `$_lastMintedFeesSharePrice` using current `totalSupply()`.
  - Returns total fee minted (after dust burn) or 0 if cooldown not met.

- `updateSpokeCaliberAccountingData($, tokenRegistry, chainRegistry, wormhole, response, signatures)`
  - Verifies CCQ response via `CaliberAccountingCCQ.decodeAndVerifyQueryResponse`.
  - For each `PerChainQueryResponse`, calls `_handlePerChainQueryResponse`.

- `_handlePerChainQueryResponse`
  - Converts Wormhole chainId to EVM chainId via `IChainRegistry.whToEvmChainId`.
  - Requires known mailbox in storage for chain; otherwise `InvalidChainId`.
  - Extracts `SpokeCaliberAccountingData` and response timestamp.
  - Validates freshness: not older than stored, and not staler than `caliberStaleThreshold`.
  - Updates net AUM, positions, base tokens, timestamp.
  - Decodes bridgesIn/Out bytes tuples, maps foreign tokens to local via `ITokenRegistry.getLocalToken`, and stores amounts in enumerable maps.

- `_getTotalAum($, oracleRegistry)`
  - For each foreign chain in `$_foreignChainIds`:
    - Ensures spoke data not stale; adds `spokeCaliberData.netAum`.
    - Checks bridge consistency in both directions (`_checkBridgeState`).
    - Adds accounting value of in-flight deltas (machine→spoke and spoke→machine not yet settled), using `_accountingValueOf`.
  - Adds hub caliber AUM via `ICaliber.getDetailedAum()`.
  - Adds idle token balances converted to accounting value.

- `_checkBridgeState(insMap, outsMap)`
  - Iterates inbound vs outbound per token amounts; if `in > out` → `BridgeStateMismatch`.

- `_accountingValueOf(oracleRegistry, accountingToken, token, amount)`
  - If token is accounting token: returns `amount`.
  - Else: gets price from `IOracleRegistry.getPrice(token, accountingToken)` and returns `amount * price / 10 ** decimals(token)`.

- `getSharePrice(aum, supply, shareTokenDecimalsOffset)`
  - Returns `SHARE_TOKEN_UNIT * (aum + 1) / (supply + 10 ** offset)`.
  - +1 and `+10**offset` avoid division-by-zero and overly spiky values at zero/near-zero states.

### Security/logic observations

- Fee accrual caps
  - Upper-bounds both fixed and performance fees with `max*AccrualRate` against `supply * elapsed / 1e18`.
  - The offset of fixed fee in perf fee base prevents double charging. Good.
  - Dependent on FeeManager implementations being non-malicious and obeying authorization (`onlyMachine` in FeeManager public API per periphery docs).

- Approve-callback pattern
  - The library approves FeeManager, calls `distributeFees`, then resets approval to 0.
  - Reentrancy exposure is minimal if Machine guards the outer call with `nonReentrant`. Confirm call sites enforce this.
  - Dust burn removes any residual minted-but-not-distributed shares, protecting supply integrity.

- CCQ verification & freshness
  - Uses guardian-signed `decodeAndVerifyQueryResponse`. Freshness checks ensure time monotonicity and non-staleness.
  - If spoke data stale, `_getTotalAum` reverts with `CaliberAccountingStale(chainId)`.

- Bridge reconciliation
  - Detects mismatches both ways and includes in-flight deltas in AUM by converting value via oracle.
  - Ensures state consistency by reverting on unexpected `in > out` mismatches.

- Decimals & pricing
  - `_accountingValueOf` scales by `10 ** DecimalsUtils._getDecimals(token)`.
  - Accuracy depends on `IOracleRegistry.getPrice` output units. Assumption: price scaled such that `amount * price / 10**decimals(token)` yields accounting token units.
  - If a token’s `decimals()` call fails and defaults to 18, conversion can mis-estimate value for non-18 tokens.

- Share price
  - `getSharePrice` guards zero cases with offsets. At tiny supply/AUM, the added constants slightly skew the value but stabilize behavior. This mirrors common ERC4626 edge handling.

### Potential risks
- Unit mismatch between Oracle price scale and token decimals could lead to valuation drift if misconfigured. Mitigated by OracleRegistry consistency.
- Default-to-18 decimals behavior may misvalue obscure tokens if Token Registry allows them. Use allowlist and tests.
- Gas growth with large numbers of chains/tokens in maps (bounded operationally; still worth monitoring).
- FeeManager external call path relies on Machine’s reentrancy guard. Ensure all entrypoints that call `manageFees` are `nonReentrant`.

### Recommendations
- Add explicit unit documentation/invariants between `IOracleRegistry.getPrice` and `_accountingValueOf` (e.g., price must be in accounting token with 18 decimals or provide `priceScale()` accessor).
- Consider caching known-decimals for supported tokens in Token Registry to avoid `decimals()` calls and fallback defaults.
- Emit events for CCQ updates and bridge reconciliation outcomes (if not already at Machine level) to improve ops observability.
- Add invariant/fuzz tests for `_getTotalAum` with randomized bridge in-flight states and varying oracle scales.

---

## Summary
- `DecimalsUtils` is simple and safe by design; the main caveat is defaulting to 18 on failures.
- `MachineUtils` encapsulates critical accounting, fee accrual, CCQ verification, and bridge reconciliation. The logic is robust with caps, freshness checks, and mismatch detection.
- The primary audit focus should be on unit consistency (oracle scales vs token decimals), reentrancy posture when calling FeeManager, and operational bounds (number of chains/tokens) impacting gas.

If you want, I can add targeted tests/fuzzers to validate `_accountingValueOf` against different oracle scaling regimes and exhaustive bridge delta scenarios.

---

## Code References (Key Excerpts)

```6:12:makina-core/src/libraries/DecimalsUtils.sol
library DecimalsUtils {
    uint8 internal constant DEFAULT_DECIMALS = 18;
    uint8 internal constant MIN_DECIMALS = 6;
    uint8 internal constant MAX_DECIMALS = DEFAULT_DECIMALS;
    uint8 internal constant SHARE_TOKEN_DECIMALS = DEFAULT_DECIMALS;
    uint256 internal constant SHARE_TOKEN_UNIT = 10 ** SHARE_TOKEN_DECIMALS;
}
```

```13:23:makina-core/src/libraries/DecimalsUtils.sol
function _getDecimals(address asset) internal view returns (uint8) {
    (bool success, bytes memory encodedDecimals) = asset.staticcall(abi.encodeCall(IERC20Metadata.decimals, ()));
    if (success && encodedDecimals.length >= 32) {
        uint256 returnedDecimals = abi.decode(encodedDecimals, (uint256));
        if (returnedDecimals <= type(uint8).max) {
            return uint8(returnedDecimals);
        }
    }
    return DEFAULT_DECIMALS;
}
```

```33:37:makina-core/src/libraries/MachineUtils.sol
function updateTotalAum(Machine.MachineStorage storage $, address oracleRegistry) external returns (uint256) {
    $._lastTotalAum = _getTotalAum($, oracleRegistry);
    $._lastGlobalAccountingTime = block.timestamp;
    return $._lastTotalAum;
}
```

```39:89:makina-core/src/libraries/MachineUtils.sol
function manageFees(Machine.MachineStorage storage $) external returns (uint256) {
    uint256 currentTimestamp = block.timestamp;
    uint256 elapsedTime = currentTimestamp - $._lastMintedFeesTime;

    if (elapsedTime >= $._feeMintCooldown) {
        address _feeManager = $._feeManager;
        address _shareToken = $._shareToken;
        uint256 currentShareSupply = IERC20(_shareToken).totalSupply();

        uint256 fixedFee = Math.min(
            IFeeManager(_feeManager).calculateFixedFee(currentShareSupply, elapsedTime),
            (currentShareSupply * elapsedTime).mulDiv($._maxFixedFeeAccrualRate, FEE_ACCRUAL_RATE_DIVISOR)
        );

        // offset fixed fee from the share price performance on which the performance fee is calculated.
        uint256 netSharePrice =
            getSharePrice($._lastTotalAum, currentShareSupply + fixedFee, $._shareTokenDecimalsOffset);
        uint256 perfFee = Math.min(
            IFeeManager(_feeManager).calculatePerformanceFee(
                currentShareSupply, $._lastMintedFeesSharePrice, netSharePrice, elapsedTime
            ),
            (currentShareSupply * elapsedTime).mulDiv($._maxPerfFeeAccrualRate, FEE_ACCRUAL_RATE_DIVISOR)
        );

        uint256 totalFee = fixedFee + perfFee;
        if (totalFee != 0) {
            uint256 balBefore = IMachineShare(_shareToken).balanceOf(address(this));

            IMachineShare(_shareToken).mint(address(this), totalFee);
            IMachineShare(_shareToken).approve(_feeManager, totalFee);

            IFeeManager(_feeManager).distributeFees(fixedFee, perfFee);

            IMachineShare(_shareToken).approve(_feeManager, 0);

            uint256 balAfter = IMachineShare(_shareToken).balanceOf(address(this));
            if (balAfter > balBefore) {
                uint256 dust = balAfter - balBefore;
                IMachineShare(_shareToken).burn(address(this), dust);
                totalFee -= dust;
            }
        }

        $._lastMintedFeesTime = currentTimestamp;
        $._lastMintedFeesSharePrice =
            getSharePrice($._lastTotalAum, IERC20(_shareToken).totalSupply(), $._shareTokenDecimalsOffset);

        return totalFee;
    }
    return 0;
}
```

```91:113:makina-core/src/libraries/MachineUtils.sol
function updateSpokeCaliberAccountingData(
    Machine.MachineStorage storage $,
    address tokenRegistry,
    address chainRegistry,
    address wormhole,
    bytes calldata response,
    GuardianSignature[] calldata signatures
) external {
    PerChainQueryResponse[] memory responses =
        CaliberAccountingCCQ.decodeAndVerifyQueryResponse(wormhole, response, signatures).responses;

    uint256 len = responses.length;
    for (uint256 i; i < len; ++i) {
        _handlePerChainQueryResponse($, tokenRegistry, chainRegistry, responses[i]);
    }
}
```

```133:144:makina-core/src/libraries/MachineUtils.sol
function getSharePrice(uint256 aum, uint256 supply, uint256 shareTokenDecimalsOffset)
    public
    pure
    returns (uint256)
{
    return DecimalsUtils.SHARE_TOKEN_UNIT.mulDiv(aum + 1, supply + 10 ** shareTokenDecimalsOffset);
}
```

```201:261:makina-core/src/libraries/MachineUtils.sol
function _getTotalAum(Machine.MachineStorage storage $, address oracleRegistry) private view returns (uint256) {
    uint256 totalAum;
    // ... per-spoke checks, stale validation, bridge state checks and in-flight deltas ...
    (uint256 hcAum,,) = ICaliber($._hubCaliber).getDetailedAum();
    totalAum += hcAum;
    // ... idle tokens converted via _accountingValueOf ...
    return totalAum;
}
```

```278:289:makina-core/src/libraries/MachineUtils.sol
function _accountingValueOf(address oracleRegistry, address accountingToken, address token, uint256 amount)
    private
    view
    returns (uint256)
{
    if (token == accountingToken) {
        return amount;
    }
    uint256 price = IOracleRegistry(oracleRegistry).getPrice(token, accountingToken);
    return amount.mulDiv(price, 10 ** DecimalsUtils._getDecimals(token));
}
```

---

## Test Coverage Snapshot (what’s covered by core tests)

- Reentrancy and role guards around flows using MachineUtils (covered; no concern)
  - Deposits: `makina-core/test/integration/concrete/machine/deposit/deposit.t.sol`
  - Redeems: `makina-core/test/integration/concrete/machine/redeem/redeem.t.sol`
  - Caliber managePosition (reentrancy + role): `.../caliber/manage-position/managePosition.t.sol`

- Accounting freshness and AUM consistency (covered; no concern)
  - Caliber accounting staleness and AUM composition: `.../caliber/get-detailed-aum/getDetailedAum.t.sol`
  - Machine AUM update cadence: `.../machine/update-total-aum/*` (AUM recompute + timestamp)

- Bridge state guardrails and mismatch detection (covered; no concern)
  - Outbound scheduling limits, chain/adapter checks, `BridgeStateMismatch`: `.../machine/transfer-to-spoke-caliber/transferToSpokeCaliber.t.sol`
  - Mailbox authorize/claim/cancel paths and roles: `.../caliber-mailbox/*`
  - Bridge controller enable/disable and loss bps: `.../bridge-controller/*`

- Migration from PreDeposit (covered; no concern)
  - Initialize with/without pre-deposit and migration invariants: `.../machine/initialize/initialize.t.sol`

- Fee accrual logic (partially covered indirectly; worth targeted tests)
  - The accrual caps, offsetting fixed fee from perf base, and dust burn are exercised when `manageFees` is invoked from Machine paths. Add focused unit tests to validate:
    - Cap enforcement at extreme `elapsedTime` and `supply` values
    - Eventual share price used in `_lastMintedFeesSharePrice`

- Decimals fallback and oracle unit assumptions (not directly covered; add tests)
  - Add unit tests for `_accountingValueOf` using tokens with non-18 decimals and failing/malformed `decimals()` to confirm valuation correctness under registry setups.

What you can safely skip worrying about (already exercised well):
- Reentrancy in user-facing and mechanic flows
- Role gating for sensitive functions
- Bridge transfer safety checks and mismatch handling
- Staleness checks on accounting data
- Pre-deposit migration invariants

Focus areas to review next (possible bug surface):
- Oracle price units vs token decimals scaling assumptions
- Fee accrual extremes and share price update timing
- Operational bounds (many chains/tokens) and gas considerations
