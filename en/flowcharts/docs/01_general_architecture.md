# Flowchart 1: General Architecture of Hyperscale-RS

## Overview

This flowchart presents the **macro view of the Hyperscale-RS system**, showing how all components connect and interact. It is the ideal starting point for understanding the overall structure of the project.

## Main Components

### Network Layer
- **P2P Network:** Peer-to-peer communication system between nodes
- Responsible for disseminating events, blocks, and votes among all nodes

### Node Layer (NodeStateMachine)
- **Central Orchestrator:** Coordinates all components of the node
- Receives events from the network and routes them to appropriate components
- Manages the lifecycle of each component

### Main Components

#### BFT State Machine
- **Protocol:** HotStuff-2 (variation of HotStuff)
- **Function:** Implement Byzantine Fault Tolerant consensus
- **Responsibilities:**
  - Manage epoch and round
  - Coordinate voting
  - Create Quorum Certificates (QC)
  - Implement Two-Chain Rule for commit

#### Execution Engine
- **Function:** Execute transactions deterministically
- **Responsibilities:**
  - Execute TX in proposed order
  - Update state (account state)
  - Generate State Root (state hash)
  - Ensure determinism

#### Mempool
- **Function:** Manage pool of pending transactions
- **Responsibilities:**
  - Receive TX from network
  - Validate TX (signature, nonce, balance)
  - Detect conflicts
  - Maintain ordered queue

#### Provisioning
- **Function:** Prepare blocks for proposal
- **Responsibilities:**
  - Select TX from mempool
  - Assemble blocks
  - Calculate Merkle Root
  - Prepare for consensus

#### Livelock Prevention
- **Function:** Detect and prevent execution cycles
- **Responsibilities:**
  - Detect cycles (TX A → B → A)
  - Defer TX to next block
  - Retry when cycle is resolved
  - Ensure progress (liveness)

### Data Layer

#### Types & Structures
- **Hash:** Unique identifiers (Blake3)
- **Block:** Block structure with TX, Merkle Root, State Root
- **Vote:** Validator vote
- **QC:** Quorum Certificate (consensus proof)

#### Cryptography
- **BLS12-381:** Aggregatable signatures (for QC)
- **Ed25519:** Fast signatures (for votes)
- **Blake3:** Cryptographic hashing

#### Storage
- **State Root:** Hash of state after execution
- **Ledger:** Immutable record of blocks
- **Persistence:** Durable storage

### Production Layer
- **Runtime:** Thread executor with specialization
- **Thread Pool Specialization:** Dedicated threads for different tasks
- Production performance optimization

## Data Flow

```
P2P Network
    ↓
NodeStateMachine (Dispatcher)
    ↓
    ├─→ BFT State Machine
    │       ├─→ Uses: Types, Cryptography
    │       └─→ Generates: QC, Votes
    │
    ├─→ Execution Engine
    │       ├─→ Uses: Types, Cryptography
    │       └─→ Generates: State Root
    │
    ├─→ Mempool
    │       ├─→ Uses: Types, Cryptography
    │       └─→ Generates: Validated TX queue
    │
    ├─→ Provisioning
    │       ├─→ Uses: Mempool, Types
    │       └─→ Generates: Prepared blocks
    │
    └─→ Livelock Prevention
            ├─→ Uses: Execution Engine
            └─→ Generates: Retry logic
    
    ↓
Storage (State Root, Ledger)
    ↓
Runtime (Thread Pool)
```

## Key Concepts

### NodeStateMachine
The heart of the system. Coordinates all components through an **event dispatcher**:

1. **Event arrives** (network, timer, TX)
2. **Dispatcher routes** to appropriate component
3. **Component processes** event
4. **Generates actions** (Vote, Propose, Execute, Broadcast)
5. **Actions update** local state
6. **Feedback** to dispatcher

### Separation of Concerns
- **BFT:** Consensus only
- **Execution:** Execution only
- **Mempool:** TX management only
- **Provisioning:** Block preparation only
- **Livelock:** Cycle prevention only

### Modular Cryptography
- **BLS12-381:** Signature aggregation (efficient for QC)
- **Ed25519:** Individual signatures (fast)
- **Blake3:** Hashing (fast and secure)

## When to Use This Flowchart

✅ **Use when:**
- You need to understand the system's general structure
- You want to know how components connect
- You are starting to learn Hyperscale-RS
- You need to explain the system to others

❌ **Don't use when:**
- You need to understand consensus details (see #02)
- You need to understand transaction flow (see #03)
- You need to understand voting (see #05)

## Next Steps

After understanding this flowchart, it is recommended to:

1. **Node State Machine (#04)** - Understand how NodeStateMachine works internally
2. **BFT Consensus Cycle (#02)** - Understand how BFT State Machine works
3. **Transaction Flow (#03)** - Understand how TX are processed

## References

- **Mermaid File:** `mermaid/01_general_architecture.mmd`
- **Image:** `images/01_general_architecture.png`
- **Repository:** https://github.com/flightofthefox/hyperscale-rs

---

**Last updated:** February 3, 2026
