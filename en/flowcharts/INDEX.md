# Hyperscale-RS Flowcharts

Complete collection of modular flowcharts that detail the operation of the Hyperscale-RS software, with emphasis on consensus cycle, epoch, block, round, and main components.

## Content

Flowcharts are organized in three complementary formats:

- **`mermaid/`** - Diagram source code (`.mmd`) - editing format
- **`images/`** - Rendered PNG images - for quick visualization
- **`docs/`** - Detailed documentation for each flowchart

## Flowchart Index

| # | Name | Description | Level | Prerequisites |
|---|------|-------------|-------|----------------|
| 01 | [General Architecture](docs/01_general_architecture.md) | Macro view of the system and its main components | Beginner | None |
| 02 | [BFT Consensus Cycle](docs/02_bft_consensus_cycle.md) | Complete consensus flow from leader election to execution | Intermediate | #01 |
| 03 | [Transaction Flow](docs/03_transaction_flow.md) | Path of a transaction from reception to finalization | Intermediate | #01, #02 |
| 04 | [Node State Machine](docs/04_node_state_machine.md) | How NodeStateMachine integrates all components | Intermediate | #01 |
| 05 | [Voting and QC Cycle](docs/05_voting_qc_cycle.md) | Detailed voting process and Quorum Certificate creation | Advanced | #02 |
| 06 | [Distributed Execution](docs/06_distributed_execution.md) | How cross-shard transactions are executed | Advanced | #03, #04 |
| 07 | [Complete Epoch Cycle](docs/07_complete_epoch_cycle.md) | Practical example with 4 validators across multiple rounds | Intermediate | #02 |
| 08 | [Failure Handling and View Change](docs/08_failure_handling_view_change.md) | How the system recovers from leader failures | Advanced | #02, #05 |

## Recommended Reading Guide

### For Beginners
1. [General Architecture](docs/01_general_architecture.md) - Understand the components
2. [BFT Consensus Cycle](docs/02_bft_consensus_cycle.md) - Understand the main flow
3. [Transaction Flow](docs/03_transaction_flow.md) - Understand TX processing
4. [Complete Epoch Cycle](docs/07_complete_epoch_cycle.md) - See everything working together

### For Intermediate Learners
1. [Node State Machine](docs/04_node_state_machine.md) - Understand integration
2. [Voting and QC Cycle](docs/05_voting_qc_cycle.md) - Security details
3. [Failure Handling and View Change](docs/08_failure_handling_view_change.md) - Understand resilience

### For Advanced Learners
1. [Distributed Execution](docs/06_distributed_execution.md) - Understand complexity
2. All flowcharts in detail
3. Repository source code

## Relationships Between Flowcharts

```
General Architecture (#01)
    ↓
Node State Machine (#04)
    ↓
    ├─→ BFT Consensus Cycle (#02)
    │       ↓
    │       ├─→ Voting and QC Cycle (#05)
    │       └─→ Failure Handling (#08)
    │
    ├─→ Transaction Flow (#03)
    │       ↓
    │       └─→ Distributed Execution (#06)
    │
    └─→ Complete Epoch Cycle (#07)
            (Integrates everything)
```

## Fundamental Concepts

### Epoch
A consensus period with a fixed set of validators. Each epoch has multiple rounds.

### Round
A consensus attempt within an epoch. Each round has a deterministically elected leader.

### Block
A structure containing transactions, merkle root, state root, and reference to the previous block.

### Quorum Certificate (QC)
Cryptographic proof that 2f+1 validators voted on a block. Contains BLS12-381 aggregated signature.

### Two-Chain Rule
A block is committed when its grandparent has a valid QC. Ensures safety (safety property).

### Vote Locking
A validator cannot vote on conflicting blocks. Ensures safety.

### Unlock Rule
Allows voting on a new block if QC is greater than previous QC. Ensures progress (liveness).

### State Root
Hash of the state after execution of all transactions. Deterministic.

### Merkle Root
Hash of all transactions in the block. Enables efficient inclusion proofs.

## References

- **Repository:** https://github.com/flightofthefox/hyperscale-rs
- **Base Protocol:** HotStuff-2 (Variation of HotStuff)
- **Cryptography:** BLS12-381 (aggregatable signatures), Ed25519 (fast signatures), Blake3 (hashing)
- **Consensus Pattern:** Byzantine Fault Tolerant (BFT)

---

**Last updated:** February 3, 2026
