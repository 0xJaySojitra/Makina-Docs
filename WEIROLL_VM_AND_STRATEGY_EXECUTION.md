# Weiroll VM and Strategy Execution - Complete Guide

This document explains Weiroll VM, instruction types, strategy execution, and how everything is managed in the Makina protocol.

## What is Weiroll VM?

**Weiroll VM** is a simple operation-chaining/scripting language for the EVM that allows executing sequences of smart contract calls in a controlled manner.

### Location and Implementation
- **Contract**: `makina-core/src/WeirollVM.sol` (wrapper around Enso's Weiroll VM)
- **Library**: `makina-core/lib/enso-weiroll/` (external dependency)
- **Interface**: `makina-core/src/interfaces/IWeirollVM.sol`

### How Weiroll VM Works

```solidity
// From makina-core/src/interfaces/IWeirollVM.sol
interface IWeirollVM {
    function execute(bytes32[] calldata commands, bytes[] memory state) 
        external returns (bytes[] memory);
}
```

**Input:**
- `commands[]`: Array of `bytes32` commands (each encodes a function call)
- `state[]`: Array of `bytes` containing initial state/parameters

**Output:**
- `bytes[]`: Final state after executing all commands

### Command Structure (bytes32)

Each command is a `bytes32` with this structure:
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───────┬─┬───────────┬─┬───────────────────────────────────────┐
│  sel  │f│    in     │o│              target                   │
└───────┴─┴───────────┴─┴───────────────────────────────────────┘
```

- `sel` (4 bytes): Function selector to call
- `f` (1 bit): Call type flag (CALL, STATICCALL, DELEGATECALL)
- `in` (6 bytes): Input indices from state array
- `o` (1 byte): Output index to store result
- `target` (20 bytes): Target contract address

### Example Command Building

```solidity
// From makina-core/test/utils/VMInstructionHelper.sol
function _buildCommand(bytes4 _selector, bytes1 _flags, bytes6 _input, bytes1 _output, address _target)
    internal pure returns (bytes32)
{
    uint256 selector = uint256(bytes32(_selector));
    uint256 flags = uint256(uint8(_flags)) << 216;
    uint256 input = uint256(uint48(_input)) << 168;
    uint256 output = uint256(uint8(_output)) << 160;
    uint256 target = uint256(uint160(_target));
    
    return bytes32(selector ^ flags ^ input ^ output ^ target);
}
```

## Instruction Types

**Location**: `makina-core/src/interfaces/ICaliber.sol` lines 29-34

```solidity
enum InstructionType {
    MANAGEMENT,        // Modifies position size
    ACCOUNTING,        // Calculates current position value
    HARVEST,          // Collects rewards from positions
    FLASHLOAN_MANAGEMENT // Modifies position within flash loan
}
```

### Instruction Structure

```solidity
// From makina-core/src/interfaces/ICaliber.sol lines 65-75
struct Instruction {
    uint256 positionId;           // ID of the position
    bool isDebt;                  // Whether position is debt
    uint256 groupId;              // Position accounting group ID
    InstructionType instructionType; // Type of instruction
    address[] affectedTokens;     // Tokens affected by instruction
    bytes32[] commands;           // Weiroll commands to execute
    bytes[] state;                // Initial state for Weiroll VM
    uint128 stateBitmap;          // Which state elements to include in hash
    bytes32[] merkleProof;        // Merkle proof for verification
}
```

## Strategy Execution Flow

### 1. Who Executes Strategies?

**Actor**: `Mechanic` role (EOA or smart contract)
- **Why EOA**: Human operators who monitor markets and execute strategies
- **Why Smart Contract**: Automated systems that can execute based on conditions

### 2. Strategy Execution Process

```solidity
// From makina-core/src/caliber/Caliber.sol
function managePosition(Instruction calldata mgmtInstruction, Instruction calldata acctInstruction)
    external
    nonReentrant
    onlyOperator
    returns (uint256, int256)
{
    // 1. Verify both instructions are allowed
    _checkInstructionIsAllowed(mgmtInstruction);
    _checkInstructionIsAllowed(acctInstruction);
    
    // 2. Execute management instruction via Weiroll VM
    IWeirollVM(weirollVm).execute(mgmtInstruction.commands, mgmtInstruction.state);
    
    // 3. Execute accounting instruction via Weiroll VM
    IWeirollVM(weirollVm).execute(acctInstruction.commands, acctInstruction.state);
    
    // 4. Update position state
    // ... position update logic
}
```

### 3. Instruction Verification Process

**Location**: `makina-core/src/caliber/Caliber.sol` lines 863-886

```solidity
function _checkInstructionIsAllowed(Instruction calldata instruction) internal {
    // 1. Hash the command sequence
    bytes32 commandsHash = keccak256(abi.encodePacked(instruction.commands));
    
    // 2. Hash the state (using bitmap to select which elements)
    bytes32 stateHash = _getStateHash(instruction.state, instruction.stateBitmap);
    
    // 3. Hash the affected tokens
    bytes32 affectedTokensHash = keccak256(abi.encodePacked(instruction.affectedTokens));
    
    // 4. Create instruction leaf hash
    bytes32 instructionLeaf = keccak256(
        abi.encode(
            keccak256(
                abi.encode(
                    commandsHash,
                    stateHash,
                    instruction.stateBitmap,
                    instruction.positionId,
                    instruction.isDebt,
                    instruction.groupId,
                    affectedTokensHash,
                    instruction.instructionType
                )
            )
        )
    );
    
    // 5. Verify Merkle proof against allowed root
    if (!MerkleProof.verify(instruction.merkleProof, _updateAllowedInstrRoot(), instructionLeaf)) {
        revert Errors.InvalidInstructionProof();
    }
}
```

## Strategy Examples

### Example 1: ERC4626 Vault Deposit (Management Instruction)

```solidity
// From makina-core/test/utils/VMInstructionHelper.sol lines 33-79
function _build4626DepositInstruction(address _caliber, uint256 _posId, address _vault, uint256 _assets)
    internal view returns (ICaliber.Instruction memory)
{
    address[] memory affectedTokens = new address[](1);
    affectedTokens[0] = IERC4626(_vault).asset(); // USDC

    bytes32[] memory commands = new bytes32[](2);
    
    // Command 1: Approve vault to spend USDC
    commands[0] = _buildCommand(
        IERC20.approve.selector,  // approve(address,uint256)
        0x01,                     // CALL
        0x0001ffffffff,          // inputs: state[0] (vault), state[1] (amount)
        0xff,                    // ignore result
        IERC4626(_vault).asset() // USDC token
    );
    
    // Command 2: Deposit to vault
    commands[1] = _buildCommand(
        IERC4626.deposit.selector, // deposit(uint256,address)
        0x01,                      // CALL
        0x0102ffffffff,           // inputs: state[1] (amount), state[2] (caliber)
        0xff,                     // ignore result
        _vault                    // vault address
    );

    bytes[] memory state = new bytes[](3);
    state[0] = abi.encode(_vault);     // vault address
    state[1] = abi.encode(_assets);    // amount to deposit
    state[2] = abi.encode(_caliber);   // recipient (caliber)

    return ICaliber.Instruction(
        _posId,                           // position ID
        false,                           // not debt
        0,                               // no group
        ICaliber.InstructionType.MANAGEMENT, // management type
        affectedTokens,                  // [USDC]
        commands,                        // [approve, deposit]
        state,                          // [vault, amount, caliber]
        0xa0000000000000000000000000000000, // state bitmap
        merkleProof                     // proof for verification
    );
}
```

### Example 2: ERC4626 Vault Accounting (Accounting Instruction)

```solidity
// From makina-core/test/utils/VMInstructionHelper.sol lines 121-174
function _build4626AccountingInstruction(address _caliber, uint256 _posId, address _vault)
    internal view returns (ICaliber.Instruction memory)
{
    address[] memory affectedTokens = new address[](1);
    affectedTokens[0] = IERC4626(_vault).asset(); // USDC

    bytes32[] memory commands = new bytes32[](3);
    
    // Command 1: Get vault asset address
    commands[0] = _buildCommand(
        IERC4626.asset.selector,  // asset()
        0x02,                     // STATICCALL
        0xffffffffffff,          // no inputs
        0x00,                    // store result at state[0]
        _vault
    );
    
    // Command 2: Get caliber's vault share balance
    commands[1] = _buildCommand(
        IERC20.balanceOf.selector, // balanceOf(address)
        0x02,                      // STATICCALL
        0x02ffffffffff,           // input: state[2] (caliber address)
        0x02,                     // store result at state[2]
        _vault                    // vault address
    );
    
    // Command 3: Preview redeem to get asset value
    commands[2] = _buildCommand(
        IERC4626.previewRedeem.selector, // previewRedeem(uint256)
        0x02,                            // STATICCALL
        0x02ffffffffff,                 // input: state[2] (share balance)
        0x00,                           // store result at state[0]
        _vault
    );

    bytes[] memory state = new bytes[](3);
    state[1] = abi.encode(ACCOUNTING_OUTPUT_STATE_END_OF_ARGS); // end marker
    state[2] = abi.encode(_caliber); // caliber address

    return ICaliber.Instruction(
        _posId,
        false,
        0,
        ICaliber.InstructionType.ACCOUNTING,
        affectedTokens,
        commands,
        state,
        0x20000000000000000000000000000000, // state bitmap
        merkleProof
    );
}
```

## How Instructions Are Managed

### 1. Merkle Tree Structure

**What**: Instructions are organized in a Merkle tree
**Where**: Root stored in `CaliberStorage._allowedInstrRoot`
**Why**: Allows efficient verification of large instruction sets

### 2. Instruction Root Updates

**Who**: `RiskManager` role (EOA or smart contract)
**Process**:
1. `RiskManager` calls `scheduleAllowedInstrRootUpdate(newRoot)`
2. Timelock delay (configurable, e.g., 24 hours)
3. After delay, new root becomes active
4. Old instructions become invalid

**Why Timelock**: Prevents sudden changes that could break strategies

### 3. Storage Locations

**Caliber Contract Storage** (`makina-core/src/caliber/Caliber.sol`):
```solidity
struct CaliberStorage {
    bytes32 _allowedInstrRoot;           // Current allowed instructions root
    bytes32 _pendingAllowedInstrRoot;    // Pending root (in timelock)
    uint256 _pendingTimelockExpiry;      // When pending root becomes active
    uint256 _timelockDuration;           // Timelock delay
    EnumerableSet.AddressSet _instrRootGuardians; // Guardians who can sign updates
}
```

**Position Storage**:
```solidity
mapping(uint256 posId => Position pos) _positionById;
mapping(uint256 groupId => EnumerableSet.UintSet positionIds) _positionIdGroups;
```

### 4. Instruction Creation Process

1. **Design Strategy**: Define what operations to perform
2. **Encode Commands**: Convert operations to Weiroll commands
3. **Build Instruction**: Create Instruction struct with all components
4. **Generate Merkle Proof**: Prove instruction is in allowed tree
5. **Submit to Tree**: Add to Merkle tree and update root
6. **Execute**: Mechanic can now execute the instruction

## Key Components Summary

| Component | Location | Purpose | Actor |
|-----------|----------|---------|-------|
| **WeirollVM** | `src/WeirollVM.sol` | Execute command sequences | Caliber contract |
| **Instruction Types** | `src/interfaces/ICaliber.sol` | Define instruction categories | Protocol design |
| **Instruction Verification** | `src/caliber/Caliber.sol` | Verify Merkle proofs | Caliber contract |
| **Strategy Execution** | `src/caliber/Caliber.sol` | Execute management + accounting | Mechanic (EOA/contract) |
| **Root Management** | `src/caliber/Caliber.sol` | Update allowed instructions | RiskManager (EOA/contract) |
| **Position Storage** | `src/caliber/Caliber.sol` | Store position data | Caliber contract |

## Why This Architecture?

1. **Security**: Merkle proofs ensure only pre-approved instructions can execute
2. **Flexibility**: Weiroll VM allows complex multi-step operations
3. **Gas Efficiency**: Batch multiple operations in single transaction
4. **Upgradeability**: New instructions can be added via root updates
5. **Separation**: Management (change position) vs Accounting (measure position) are separate
6. **Timelock Safety**: Prevents sudden instruction changes that could break strategies

This system allows the Makina protocol to execute sophisticated DeFi strategies while maintaining security through cryptographic verification of allowed operations.
