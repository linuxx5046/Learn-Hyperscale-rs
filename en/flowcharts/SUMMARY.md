# Flowcharts Summary - Hyperscale-RS

## Structured Overview

| # | Name | Objective | Main Components | Complexity | Reading Time |
|---|------|-----------|-----------------|------------|--------------|
| 01 | General Architecture | System macro view | Layers: Network, Node, Components, Data, Production | ⭐ Beginner | 5 min |
| 02 | BFT Consensus Cycle | Complete consensus flow | Epoch, Round, Leader, Proposal, Voting, QC, Execution | ⭐⭐ Intermediate | 15 min |
| 03 | Transaction Flow | TX path to finalization | Reception, Validation, Mempool, Proposal, Consensus, Execution | ⭐⭐ Intermediate | 10 min |
| 04 | Node State Machine | Component integration | BFT State, Execution State, Mempool, Provisioning, Livelock | ⭐⭐ Intermediate | 12 min |
| 05 | Voting and QC Cycle | Detailed voting process | Vote Locking, Broadcast, Collection, Quorum, BLS Aggregation | ⭐⭐⭐ Advanced | 20 min |
| 06 | Distributed Execution | Cross-shard transactions | Intra-Shard, Cross-Shard, Prepare-Commit, Livelock Prevention | ⭐⭐⭐ Advanced | 18 min |
| 07 | Complete Epoch Cycle | Practical example with validators | 4 Validators, multiple Rounds, Two-Chain Rule, Commit | ⭐⭐ Intermediate | 12 min |
| 08 | Failure Handling and View Change | Leader failure recovery | Timeout, Unlock, View Change, New Leader, Synchronization | ⭐⭐⭐ Advanced | 15 min |

## Recommended Learning Paths

### Beginner Path (45 minutes)
```
01. General Architecture (5 min)
    ↓
02. BFT Consensus Cycle (15 min)
    ↓
03. Transaction Flow (10 min)
    ↓
07. Complete Epoch Cycle (12 min)
```

### Intermediate Path (90 minutes)
```
01. General Architecture (5 min)
    ↓
04. Node State Machine (12 min)
    ↓
02. BFT Consensus Cycle (15 min)
    ↓
03. Transaction Flow (10 min)
    ↓
07. Complete Epoch Cycle (12 min)
    ↓
08. Failure Handling (15 min)
    ↓
05. Voting and QC Cycle (20 min)
```

### Advanced Path (120+ minutes)
```
All flowcharts in order:
01 → 04 → 02 → 03 → 07 → 05 → 08 → 06
(with deep source code reading)
```

## Concepts by Flowchart

### 01 - General Architecture
**Concepts:** Layers, Components, Orchestration
- NodeStateMachine (orchestrator)
- BFT State Machine
- Execution Engine
- Mempool
- Provisioning
- Livelock Prevention
- Cryptography (BLS12-381, Ed25519, Blake3)

### 02 - BFT Consensus Cycle
**Concepts:** Epoch, Round, Consensus, Execution
- Epoch
- Round
- Leader Election
- Block Proposal
- Voting (2f+1)
- Quorum Certificate (QC)
- Two-Chain Rule
- Deterministic Execution

### 03 - Transaction Flow
**Concepts:** TX Pipeline, Mempool, Finalization
- TX Reception
- Validation (signature, nonce, balance)
- Mempool (queue, conflict detection)
- Proposal (TX selection)
- Consensus (voting, QC)
- Execution (deterministic)
- Finalization (immutability)

### 04 - Node State Machine
**Concepts:** Integration, Dispatcher, Event Loop
- BFT State (Epoch, Round, View)
- Execution State (Account State)
- Mempool State (pending TXs)
- Provisioning State (Blocks)
- Livelock State (Cycle Detection)
- Event Dispatcher
- Action Generation

### 05 - Voting and QC Cycle
**Concepts:** Safety, Aggregation, Verification
- Vote Locking (safety)
- Unlock Rule (liveness)
- Ed25519 Signature
- Batch Verification
- BLS12-381 Aggregation
- Quorum Certificate
- QC Validation

### 06 - Distributed Execution
**Concepts:** Sharding, Cross-Shard, Deadlock Prevention
- Intra-Shard (direct execution)
- Cross-Shard (prepare-commit)
- CommitmentProof
- Livelock Prevention
- Cycle Detection
- Retry Logic

### 07 - Complete Epoch Cycle
**Concepts:** Practical Example, Multiple Rounds
- 4 Validators
- Fault Tolerance (f=1)
- Quorum (2f+1 = 3)
- Multiple Rounds
- Two-Chain Rule
- Commit

### 08 - Failure Handling and View Change
**Concepts:** Resilience, Recovery, Liveness
- Timeout Detection
- Vote Unlock
- View Change
- New Leader
- Synchronization
- HighQC
- Recovery

## Dependency Map

```
No dependencies
    ↓
01. General Architecture
    ↓
    ├─→ 04. Node State Machine (requires 01)
    │       ↓
    │       ├─→ 02. Consensus BFT (requires 01, 04)
    │       │       ↓
    │       │       ├─→ 05. Voting and QC (requires 02)
    │       │       └─→ 08. Failures (requires 02, 05)
    │       │
    │       ├─→ 03. TX Flow (requires 01, 02)
    │       │       ↓
    │       │       └─→ 06. Distributed Execution (requires 03, 04)
    │       │
    │       └─→ 07. Complete Cycle (requires 02)
    │
    └─→ All flowcharts can be consulted after 01
```

## Usage Guide by Profile

### New Developer in Project
1. Read 01 (Architecture)
2. Read 02 (Consensus)
3. Read 03 (Transactions)
4. Consult 05 and 08 as needed

### Researcher / Academic
1. Read all in order
2. Focus on 02, 05, 08 (safety and resilience)
3. Study 06 (distributed execution)

### Code Contributor
1. Read 01 (understand architecture)
2. Read 04 (understand integration)
3. Read flowcharts specific to your component

### Security Auditor / Reviewer
1. Read 02 (consensus)
2. Read 05 (voting and safety)
3. Read 08 (resilience)
4. Read 06 (distributed execution)

## Statistics

- **Total Flowcharts:** 8
- **Total Reading Time:** ~100 minutes
- **Beginner Level:** 1 flowchart (5 min)
- **Intermediate Level:** 4 flowcharts (47 min)
- **Advanced Level:** 3 flowcharts (53 min)
- **Available Formats:** Mermaid (.mmd), PNG (.png), Documentation (.md)

---

**Last updated:** February 3, 2026
