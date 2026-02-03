# Análise da Arquitetura Hyperscale-rs

## Visão Geral do Projeto

**Hyperscale-rs** é uma implementação em Rust de um protocolo de consenso distribuído de alta performance, baseado em **HotStuff-2** com otimizações para sistemas de sharding (fragmentação de estado).

### Estatísticas do Projeto
- **Linhas de código**: ~76.477 linhas de Rust
- **Número de crates**: 16 crates especializados
- **Arquitetura**: Modular, baseada em máquinas de estado síncronas
- **Foco**: Determinismo, testabilidade, performance em produção

---

## Estrutura de Crates (Dependências Hierárquicas)

### Camada 1: Tipos Fundamentais
```
hyperscale-types (6.988 linhas)
├─ Primitivos: Hash, Keys (Ed25519, BLS12-381)
├─ Identifiers: ValidatorId, BlockHeight, ShardGroupId
├─ Consensus: Block, BlockHeader, QuorumCertificate
├─ Transactions: RoutableTransaction, TransactionCertificate
└─ Topology: Validator sets, shard configuration
```

**Características Principais:**
- Sem dependências de outros crates do workspace
- Usa **BLS12-381** para assinaturas agregáveis (QCs)
- Usa **Ed25519** para assinaturas rápidas (transações)
- Merkle trees com tagged leaves para provas de posição

### Camada 2: Abstrações Core
```
hyperscale-core (2.126 linhas)
├─ StateMachine trait (síncrono, determinístico)
├─ Event enum (entrada do sistema)
├─ Action enum (saída do sistema)
└─ StateRootComputer trait (JMT integration)
```

**Filosofia:**
- **Síncrono**: Sem async/await
- **Determinístico**: Mesmo estado + evento = mesmas ações
- **Puro**: Sem I/O (tudo retornado como Actions)

### Camada 3: Máquinas de Estado Especializadas

#### BFT Consensus (10.801 linhas)
```
hyperscale-bft
├─ BftState (máquina de estado principal)
├─ Proposal: block proposal e broadcasting
├─ Voting: vote collection e QC formation
├─ View Changes: implicit (HotStuff-2)
└─ Vote Locking: safety critical
```

**Protocolo HotStuff-2:**
- **Two-chain commit**: Bloco em altura H é commitado quando QC forma em H+1
- **Implicit view changes**: Timeout local, sem coordenação
- **Vote locking**: Previne equivocação (safety)
- **Unlock rule**: QC em altura H desbloqueia votos em ≤H

#### Execution (3.464 linhas)
```
hyperscale-execution
├─ ExecutionState (coordenação de execução)
├─ Single-shard execution
├─ Cross-shard coordination (2PC)
└─ State provisioning
```

#### Mempool (3.132 linhas)
```
hyperscale-mempool
├─ Transaction submission
├─ Gossip coordination
├─ Conflict detection
└─ Status tracking
```

#### Livelock Prevention (1.294 linhas)
```
hyperscale-livelock
├─ Cycle detection (cross-shard)
├─ Deadlock prevention
└─ Deferral coordination
```

#### Provisions (1.580 linhas)
```
hyperscale-provisions
├─ Centralized provision coordination
├─ Cross-shard transaction support
└─ State consistency
```

### Camada 4: Composição
```
hyperscale-node (1.109 linhas)
└─ NodeStateMachine
   ├─ BftState
   ├─ ExecutionState
   ├─ MempoolState
   ├─ ProvisionCoordinator
   └─ LivelockState
```

### Camada 5: Simulação e Produção

#### Simulação (4.862 linhas)
```
hyperscale-simulation
├─ SimulationRunner (determinístico)
├─ Event Queue (BTreeMap ordenado)
├─ SimStorage (em memória)
├─ SimulatedNetwork (latência configurável)
└─ NetworkTrafficAnalyzer
```

**Características:**
- Completamente determinístico
- Mesmo seed = mesmos resultados
- Suporta latência artificial
- Análise de tráfego de rede

#### Produção (21.965 linhas)
```
hyperscale-production
├─ ProductionRunner (async/tokio)
├─ Libp2p networking
├─ RocksDB storage
├─ Thread pools especializados
│  ├─ Crypto pool (rayon)
│  ├─ Execution pool (rayon)
│  └─ I/O pool (tokio)
├─ SyncManager
├─ ValidationBatcher
└─ Telemetry (Prometheus)
```

**Arquitetura de Threads:**
```
Core 0 (pinned): State Machine + Event Loop
    ↓
    ├→ Crypto Pool (rayon): BLS verification
    ├→ Execution Pool (rayon): Radix Engine
    └→ I/O Pool (tokio): Network, Storage, Timers
```

---

## Fluxo de Dados (Event-Driven)

### Ciclo Principal
```
1. Event entra no StateMachine
   ↓
2. StateMachine.handle(event) → Vec<Action>
   ↓
3. Runner executa Actions
   ├─ SendMessage → network
   ├─ SetTimer → timers
   ├─ VerifyCrypto → crypto pool
   ├─ ExecuteTransaction → execution pool
   ├─ CommitState → storage
   └─ ...
   ↓
4. Resultados de Actions → novos Events
   ↓
5. Volta ao passo 1
```

### Exemplo: Consenso de um Bloco

```
ProposalTimer
    ↓
BftState.on_proposal_timer()
    → Action::ProposeBlock
    ↓
Runner broadcasts block header
    ↓
BlockHeaderReceived (em todos os nós)
    ↓
BftState.on_block_header()
    → Validate, assemble, vote
    → Action::SendBlockVote
    ↓
BlockVoteReceived (agregação em proposer)
    ↓
BftState.on_block_vote()
    → Coleta votos, forma QC
    → Action::QuorumCertificateFormed
    ↓
QuorumCertificateFormed
    ↓
BftState.on_qc_formed()
    → Atualiza chain state
    → Commit se ready (two-chain)
    → Action::CommitBlock
```

---

## Componentes Críticos para Segurança

### 1. Criptografia
- **BLS12-381** (G1 private, G2 signatures)
  - Assinaturas agregáveis (QCs)
  - Batch verification
  - Verificação de ciclos (CycleProof)
  
- **Ed25519**
  - Assinaturas rápidas (transações)
  - Batch verification
  - Notarização de transações

### 2. Consenso (HotStuff-2)
- **Vote Locking**: Previne equivocação
  - Rastreado em `voted_heights: HashMap<u64, (Hash, u64)>`
  - Persistido em storage (safety critical)
  
- **Quorum Intersection**: 2f+1 de n=3f+1
  - Qualquer dois quorums sobrepõem em ≥1 validador honesto
  - Impede conflitos
  
- **Two-Chain Commit**: Finality rápida
  - Bloco H commitado quando QC forma em H+1
  - Garante finality mesmo sob assincronia

### 3. Execução Distribuída
- **Cross-Shard 2PC**: Two-Phase Commit
  - Coordenação via provisions
  - Deferral para prevenir deadlock
  
- **State Provisioning**: Consistência
  - JMT (Jellyfish Merkle Tree) state roots
  - Verificação de state root antes de voting
  - Especulative execution com rollback

### 4. Livelock Prevention
- **Cycle Detection**: Detecta ciclos cross-shard
- **Deferral**: Adia transações problemáticas
- **Retry**: Retenta após resolução

---

## Padrões de Software em Produção

### 1. State Machine Pattern
- Toda lógica é síncrona e determinística
- Facilita testes, simulação, debugging
- Separação clara entre lógica e I/O

### 2. Event Aggregator Pattern
- Single task owns state machine
- Events via mpsc channel
- Evita mutex contention

### 3. Thread Pool Specialization
- **Crypto pool**: Operações criptográficas
- **Execution pool**: Radix Engine (CPU-bound)
- **I/O pool**: Network, storage, timers
- Cada pool otimizado para seu workload

### 4. Deterministic Simulation
- Mesmo seed = mesmos resultados
- Facilita debugging de race conditions
- Testes reproduzíveis

### 5. Batch Processing
- Batch verification de assinaturas
- Batch validation de transações
- Reduz overhead de context switching

---

## Fluxo de Consenso Detalhado

### Fase 1: Proposal
```
Proposer (round-robin):
  1. Seleciona transações do mempool
  2. Executa especulativamente
  3. Computa state root (JMT)
  4. Cria BlockHeader
  5. Assina header (BLS)
  6. Broadcast BlockHeader
  7. Aguarda assembling de dados
```

### Fase 2: Voting
```
Validator recebe BlockHeader:
  1. Valida assinatura do proposer
  2. Aguarda dados completos (gossip/fetch)
  3. Valida estado (state root)
  4. Valida transações
  5. Verifica vote lock (safety)
  6. Cria BlockVote (BLS)
  7. Envia para proposer
```

### Fase 3: QC Formation
```
Proposer coleta votos:
  1. Valida cada voto (BLS)
  2. Aguarda 2f+1 votos
  3. Agrega assinaturas (BLS aggregation)
  4. Cria QuorumCertificate
  5. Broadcast QC
  6. Atualiza chain state
```

### Fase 4: Commitment
```
Quando QC forma para altura H+1:
  1. Bloco em altura H é commitado
  2. Executa transações (se não foi especulativo)
  3. Atualiza JMT state
  4. Persiste em RocksDB
  5. Avança altura
```

---

## Tipos de Dados Principais

### Block Structure
```rust
Block {
    header: BlockHeader {
        height: u64,
        parent_hash: Hash,
        parent_qc: QuorumCertificate,
        proposer: ValidatorId,
        timestamp: u64,
        round: u64,
        is_fallback: bool,
        state_root: Hash,
        state_version: u64,
        transaction_root: Hash,
    },
    retry_transactions: Vec<Arc<RoutableTransaction>>,
    priority_transactions: Vec<Arc<RoutableTransaction>>,
    transactions: Vec<Arc<RoutableTransaction>>,
    certificates: Vec<Arc<TransactionCertificate>>,
    deferred: Vec<TransactionDefer>,
    aborted: Vec<TransactionAbort>,
}
```

### Transaction Types
```rust
RoutableTransaction {
    tx: NotarizedTransactionV1,  // Radix Engine format
    read_nodes: Vec<NodeId>,     // Shards lidos
    write_nodes: Vec<NodeId>,    // Shards escritos
}

TransactionCertificate {
    tx_hash: Hash,
    decision: TransactionDecision,  // Accept/Reject
    state_writes: Vec<SubstateWrite>,
}
```

### Quorum Certificate
```rust
QuorumCertificate {
    block_hash: Hash,
    height: BlockHeight,
    round: u64,
    signer_bitfield: SignerBitfield,  // Quem assinou
    aggregated_signature: Bls12381G2Signature,
}
```

---

## Configuração de Topologia

### Sharding
- **Múltiplos shards**: Cada shard tem seu próprio validador set
- **Cross-shard transactions**: Coordenadas via 2PC
- **Provisions**: Coordenação centralizada por shard

### Validadores
- **Round-robin proposer**: `proposer = (height + round) % num_validators`
- **Quorum**: 2f+1 de n=3f+1 (tolera f Byzantine)
- **Topology**: Dinâmica (pode mudar entre épocas)

---

## Próximas Seções Necessárias

1. **Criptografia Detalhada**
   - BLS12-381 e agregação
   - Ed25519 e batch verification
   - Merkle trees e provas

2. **Consenso Distribuído Detalhado**
   - HotStuff-2 vs HotStuff original
   - Implicit view changes
   - Safety e liveness proofs

3. **Execução Distribuída**
   - 2PC para cross-shard
   - JMT (Jellyfish Merkle Tree)
   - State provisioning

4. **Padrões de Produção**
   - Thread pool management
   - Batch processing
   - Telemetry e monitoring

5. **Exemplos de Código**
   - Como estender o sistema
   - Testes e simulação
   - Debugging



---

# Parte 2: Criptografia em Detalhes

## Algoritmos Criptográficos Utilizados

### 1. Blake3 para Hashing
```rust
// Usado em: Hashes de blocos, transações, merkle trees
pub fn hash_from_bytes(bytes: &[u8]) -> Hash {
    let hash = blake3::hash(bytes);
    Hash(*hash.as_bytes())
}

// Merkle tree com Blake3
pub fn compute_merkle_root(hashes: &[Hash]) -> Hash {
    // Combina pares de hashes em cada nível
    // Nós ímpares promovem unchanged para o próximo nível
}
```

**Características:**
- 32 bytes (256 bits) de output
- Determinístico (mesmo input = mesmo output)
- Resistência a colisões
- Rápido para computação em massa

### 2. BLS12-381 para Assinaturas Agregáveis
```rust
// Usado em: Quorum Certificates, CycleProofs, StateProvisions
pub struct Bls12381G1PrivateKey { /* ... */ }
pub struct Bls12381G2Signature { /* ... */ }

// Agregação de assinaturas
let aggregated_sig = aggregate_signatures(&[sig1, sig2, sig3]);
```

**Características:**
- **G1 Private Keys**: Escalares do campo BLS12-381
- **G2 Signatures**: Pontos na curva G2
- **Agregação**: Múltiplas assinaturas → uma única assinatura
- **Batch Verification**: Verifica múltiplas assinaturas em paralelo
- **Domain Separation**: Cada tipo de mensagem tem tag único

**Uso no Hyperscale:**
- QCs agregam 2f+1 votos em uma única assinatura (48 bytes)
- CommitmentProofs agregam provisions de múltiplos validadores
- CycleProofs agregam StateProvisions

### 3. Ed25519 para Assinaturas Rápidas
```rust
// Usado em: Notarização de transações, assinaturas individuais
pub struct Ed25519PrivateKey { /* ... */ }
pub struct Ed25519Signature { /* ... */ } // 64 bytes
```

**Características:**
- Mais rápido que BLS12-381 para assinatura/verificação individual
- Não agregável (cada assinatura é independente)
- Batch verification otimizado
- Usado para transações (não crítico para consenso)

## Domain Separation

**Conceito**: Previne ataques de replay onde uma assinatura de um contexto é reutilizada em outro.

```rust
// Cada tipo de mensagem tem domain tag único
pub const DOMAIN_BLOCK_VOTE: &[u8] = b"BLOCK_VOTE";
pub const DOMAIN_STATE_PROVISION: &[u8] = b"STATE_PROVISION";
pub const DOMAIN_EXEC_VOTE: &[u8] = b"EXEC_VOTE";

// Mensagem assinada = DOMAIN_TAG || conteúdo
fn block_vote_message(
    shard_group: ShardGroupId,
    height: u64,
    round: u64,
    block_hash: &Hash,
) -> Vec<u8> {
    let mut msg = Vec::new();
    msg.extend_from_slice(DOMAIN_BLOCK_VOTE);
    msg.extend_from_slice(&shard_group.0.to_le_bytes());
    msg.extend_from_slice(&height.to_le_bytes());
    msg.extend_from_slice(&round.to_le_bytes());
    msg.extend_from_slice(block_hash.as_bytes());
    msg
}
```

**Benefícios:**
- Assinatura para BLOCK_VOTE não pode ser reutilizada para STATE_PROVISION
- Cada shard tem seu próprio domínio (shard_group_id)
- Cada altura e round têm seu próprio domínio

## Batch Verification

**Problema**: Verificar 2f+1 assinaturas individualmente é lento.

**Solução**: Batch verification agrupa verificações.

```rust
// Ed25519 batch verification
pub fn batch_verify_ed25519(
    messages: &[&[u8]],
    signatures: &[Ed25519Signature],
    pubkeys: &[Ed25519PublicKey],
) -> bool {
    // Usa ed25519-dalek batch verification
    // ~2x mais rápido que verificação individual para batches de 64+
}

// BLS12-381 batch verification (via Action::VerifyQcSignature)
// Delegado ao runner para execução em thread pool de crypto
```

**Aplicações:**
- Verificação de QC (2f+1 votos agregados)
- Verificação de votos em lote
- Verificação de StateProvisions

## Merkle Trees com Tagged Leaves

**Problema**: Como provar que uma transação está em um bloco E qual é sua posição?

**Solução**: Tagged leaves com prefixos.

```rust
// Cada transação é tagged por sua seção
const RETRY_TAG: &[u8] = b"RETRY";
const PRIORITY_TAG: &[u8] = b"PRIORITY";
const NORMAL_TAG: &[u8] = b"NORMAL";

// Leaf hash = hash(TAG || tx_hash)
let retry_leaf = Hash::from_parts(&[RETRY_TAG, tx_hash.as_bytes()]);

// Merkle root = hash de todas as leaves
pub fn compute_transaction_root(
    retry_transactions: &[Arc<RoutableTransaction>],
    priority_transactions: &[Arc<RoutableTransaction>],
    transactions: &[Arc<RoutableTransaction>],
) -> Hash {
    // Concatena todas as tagged leaves
    // Computa merkle root
}
```

**Benefícios:**
- Prova de inclusão (merkle path)
- Prova de posição (via tag no path)
- Prova de ordenação (via merkle tree structure)

---

# Parte 3: Consenso Distribuído Detalhado

## HotStuff-2 vs HotStuff Original

### HotStuff Original (3 rounds para commit)
```
Round 1 (Prepare): Proposer → Validators vote
Round 2 (Pre-commit): Collect votes → Form QC
Round 3 (Commit): Broadcast QC → Validators commit
```

### HotStuff-2 (2 rounds para commit)
```
Round 1: Proposer → Validators vote
Round 2: Collect votes → Form QC → Commit (two-chain rule)

// Bloco em altura H é commitado quando QC forma em H+1
// Não precisa de round 3 separado
```

**Benefício**: Latência reduzida de 3 rounds para 2 rounds.

## Vote Locking (Safety Critical)

**Invariante de Segurança**: Um validador NUNCA pode votar em dois blocos diferentes na mesma altura.

```rust
// Rastreado em voted_heights
pub voted_heights: HashMap<u64, (Hash, u64)>

// Quando votamos em um bloco:
if let Some(&(existing_hash, _)) = self.voted_heights.get(&height) {
    if existing_hash != block_hash {
        // Violação de safety - não votamos
        return vec![];
    }
}

// Persistido em storage (recovery critical)
pub struct RecoveredState {
    pub voted_heights: HashMap<u64, (Hash, u64)>,
    // ...
}
```

**Por que é crítico:**
- Previne equivocação (Byzantine validator votando em múltiplos blocos)
- Garante que qualquer dois quorums (2f+1) se sobrepõem em ≥1 validador honesto
- Impede conflitos na chain

## Unlock Rule (Liveness)

**Problema**: Vote locking pode travar consenso se todos os validadores estão locked em blocos diferentes.

**Solução**: Unlock quando vemos QC em altura H.

```rust
// Quando recebemos QC em altura H
fn maybe_unlock_for_qc(&mut self, qc: &QuorumCertificate) {
    let qc_height = qc.height.0;
    
    // Remover locks em alturas ≤ qc_height
    self.voted_heights.retain(|&height, _| height > qc_height);
}
```

**Como funciona:**
1. Validador A vota em bloco B1 em altura 10
2. Validador B vota em bloco B2 em altura 10 (diferente)
3. Nenhum atinge quorum
4. View change para round 1
5. Novo proposer propõe bloco B3 em altura 10
6. Validador A recebe QC de altura 9
7. Unlock ocorre → A pode votar em B3
8. Quorum formado → Consenso avança

## View Changes (Implicit)

**HotStuff-2 usa view changes implícitos**: Cada validador avança seu round localmente no timeout.

```rust
// Timeout local (sem coordenação)
pub fn on_proposal_timer(&mut self) -> Vec<Action> {
    // Se proposer não produziu bloco em tempo
    // Avançar round localmente
    self.view += 1;
    self.view_at_height_start = self.view;
    
    // Próximo proposer = (height + new_round) % num_validators
    // Proposer muda automaticamente
}
```

**Benefícios:**
- Sem protocolo de view change separado
- Sem mensagens de view change
- Cada validador avança independentemente
- Sincronização via QCs (quando vemos QC em round R, avançamos para R)

## Two-Chain Commit Rule

**Regra**: Bloco em altura H é commitado quando QC forma em altura H+1.

```rust
// Quando QC forma em altura H+1
fn on_qc_formed(&mut self, qc: &QuorumCertificate) {
    let qc_height = qc.height.0;
    
    // Bloco em qc_height - 1 está agora commitado
    if qc_height > 0 {
        let committed_height = qc_height - 1;
        // Commit block at committed_height
    }
}
```

**Por que funciona:**
- QC em H+1 prova que ≥2f+1 validadores viram bloco em H
- Qualquer conflito em H seria detectado
- Finality garantida mesmo sob assincronia

## State Root Verification

**Problema**: Como validadores verificam que o proposer computou o state_root correto?

**Solução**: Verificação assíncrona com JMT (Jellyfish Merkle Tree).

```rust
// Proposer computa state_root especulativamente
fn build_proposal(&mut self) -> Vec<Action> {
    // Coleta certificados do mempool
    let certificates = self.get_certificates();
    
    // Computa state_root especulativamente
    let state_root = self.compute_speculative_root(
        parent_state_root,
        &certificates,
    );
    
    // Inclui no header
    BlockHeader {
        state_root,
        state_version,
        // ...
    }
}

// Validador verifica state_root antes de votar
fn verify_state_root(&mut self, block: &Block) -> Vec<Action> {
    // Aguarda JMT estar pronto
    // Computa state_root localmente
    // Compara com block.header.state_root
    // Se match → vote
    // Se mismatch → reject
}
```

**Critical Issue Evitado**: Deadlock de state_root

```
// BUG: Usar parent's speculative state_version
Block A (height 10): 5 certs, state_version = 9
Block B (height 11): 40 certs, state_version = 49 (speculative, not committed)
Block C (height 12): 10 certs, state_version = 59 (extends B)

Verifier needs JMT at version 49 to verify C
But JMT is at version 9 (only A committed)
DEADLOCK!

// FIX: Usar committed state_version
Block A (height 10): 5 certs, state_version = 9 (committed)
Block B (height 11): 40 certs, state_version = 49 (speculative)
Block C (height 12): 10 certs, state_version = 59 (extends B)

Use committed version (9) as base for C
Verifier can compute: 9 + 10 = 19
No deadlock!
```

---

# Parte 4: Execução Distribuída

## Cross-Shard Coordination (2PC)

**Problema**: Transação lê/escreve em múltiplos shards. Como garantir atomicidade?

**Solução**: Two-Phase Commit (2PC) com provisions.

### Fase 1: Prepare (Source Shard)
```
1. Source shard executa transação
2. Gera StateProvisions (estado que target shard precisa)
3. Assina provisions com BLS (StateProvision domain)
4. Envia para target shard
```

### Fase 2: Commit (Target Shard)
```
1. Target shard recebe StateProvisions
2. Valida assinaturas (batch verification)
3. Agrega em CommitmentProof (BLS aggregation)
4. Inclui CommitmentProof em bloco
5. Transação pode ser executada no target shard
```

## Livelock Prevention

**Problema**: Ciclos cross-shard causam deadlock.

```
TX A: Shard 0 → Shard 1
TX B: Shard 1 → Shard 0

Shard 0 aguarda provision de B (que está em Shard 1)
Shard 1 aguarda provision de A (que está em Shard 0)
DEADLOCK!
```

**Solução**: Cycle Detection + Deferral

```rust
// Detectar ciclo
fn detect_cycle(tx: &Transaction) -> Option<CycleProof> {
    // Verificar se há caminho de volta
    // Se sim, gerar CycleProof (assinado por quorum)
}

// Adiar transação
fn defer_transaction(tx: &Transaction, proof: CycleProof) {
    // Incluir em bloco como TransactionDefer
    // Retornar com novo hash (retry)
    // Tentar novamente em bloco futuro
}
```

**CycleProof**: Prova que ciclo foi detectado por quorum de validadores.

```rust
pub struct CycleProof {
    // Qual transação ganhou (será executada)
    pub winner_tx_hash: Hash,
    
    // Qual transação perdeu (será adiada)
    pub loser_tx_hash: Hash,
    
    // CommitmentProof do winner (prova que foi commitado)
    pub winner_commitment: CommitmentProof,
    
    // Assinado por quorum de source shard
    pub aggregated_signature: Bls12381G2Signature,
}
```

---

# Parte 5: Padrões de Software em Produção

## State Machine Pattern

**Conceito**: Toda lógica é síncrona, determinística, sem I/O.

```rust
pub trait StateMachine {
    fn handle(&mut self, event: Event) -> Vec<Action>;
    fn set_time(&mut self, now: Duration);
    fn now(&self) -> Duration;
}

// Implementação
impl StateMachine for NodeStateMachine {
    fn handle(&mut self, event: Event) -> Vec<Action> {
        match event {
            Event::ProposalTimer => self.bft.on_proposal_timer(),
            Event::BlockHeaderReceived { header, ... } => {
                self.bft.on_block_header(header, ...)
            }
            // ...
        }
    }
}
```

**Benefícios:**
- Testável (sem dependências externas)
- Determinístico (mesmo estado + evento = mesmas ações)
- Simulável (roda em simulação determinística)
- Debugável (trace completo de eventos)

## Event Aggregator Pattern (Produção)

```rust
// Single task owns state machine
async fn run_state_machine(
    mut state_machine: NodeStateMachine,
    mut event_rx: mpsc::Receiver<Event>,
) {
    loop {
        // Receber evento
        let event = event_rx.recv().await;
        
        // Processar (síncrono)
        let actions = state_machine.handle(event);
        
        // Executar actions (I/O)
        for action in actions {
            execute_action(action).await;
        }
    }
}

// Múltiplos produtores de eventos
// Network → mpsc channel → Event Aggregator
// Timers → mpsc channel → Event Aggregator
// Storage → mpsc channel → Event Aggregator
```

**Benefícios:**
- Sem mutex (single owner)
- Sem contention (serial processing)
- Sem race conditions (eventos ordenados)

## Thread Pool Specialization (Produção)

```rust
pub struct ThreadPoolManager {
    // Crypto pool: BLS verification, signature checks
    crypto_pool: rayon::ThreadPool,
    
    // Execution pool: Radix Engine, merkle computation
    execution_pool: rayon::ThreadPool,
    
    // I/O pool: tokio runtime for network/storage/timers
    io_runtime: tokio::runtime::Runtime,
}

// Dispatch actions to appropriate pool
match action {
    Action::VerifyQcSignature { ... } => {
        crypto_pool.spawn(|| verify_qc_signature(...));
    }
    Action::ExecuteTransaction { ... } => {
        execution_pool.spawn(|| execute_transaction(...));
    }
    Action::SendMessage { ... } => {
        io_runtime.spawn(async { send_message(...).await });
    }
}
```

**Configuração de Threads**:
```rust
// Auto-detect cores
let config = ThreadPoolConfig::auto();
// 25% crypto, 50% execution, 25% I/O

// Ou customizar
let config = ThreadPoolConfig::builder()
    .crypto_threads(4)
    .execution_threads(8)
    .io_threads(2)
    .pin_cores(true)  // Linux only
    .build()?;
```

## Batch Processing

### Batch Verification de Assinaturas
```rust
// Ao invés de verificar cada voto individualmente:
for vote in votes {
    verify_signature(&vote)?;  // Lento
}

// Batch verify quando temos quorum:
Action::VerifyAndBuildQuorumCertificate {
    votes,
    public_keys,
    // Runner verifica todos em paralelo
}
```

### Batch Validation de Transações
```rust
// Validar múltiplas transações em paralelo
let validation_results = validation_pool.par_iter(transactions)
    .map(|tx| validate_transaction(tx))
    .collect();
```

## Deterministic Simulation

```rust
pub struct SimulationRunner {
    // Event queue ordenado por: (time, priority, node, sequence)
    event_queue: BTreeMap<EventKey, Event>,
    
    // Nodes (in-process)
    nodes: Vec<NodeStateMachine>,
    
    // Storage (in-memory)
    storage: SimStorage,
    
    // Network (simulated latency)
    network: SimulatedNetwork,
}

// Determinístico: mesmo seed = mesmos resultados
let mut runner = SimulationRunner::new(config, seed=42);
runner.initialize_genesis();
runner.run_until(Duration::from_secs(10));

// Trace completo de eventos
for event in runner.event_log() {
    println!("{:?}", event);
}
```

**Uso:**
- Testes de consenso
- Debugging de race conditions
- Verificação de liveness
- Análise de performance

---

# Parte 6: Estrutura de Dados Principais

## PendingBlock (Assembling de Blocos)

```rust
pub struct PendingBlock {
    // Header (sempre presente)
    header: BlockHeader,
    
    // Hashes de transações (sempre presentes)
    retry_hashes: Vec<Hash>,
    priority_hashes: Vec<Hash>,
    tx_hashes: Vec<Hash>,
    
    // Dados reais (podem estar faltando)
    retry_transactions: HashMap<Hash, Arc<RoutableTransaction>>,
    priority_transactions: HashMap<Hash, Arc<RoutableTransaction>>,
    transactions: HashMap<Hash, Arc<RoutableTransaction>>,
    
    // Certificados (podem estar faltando)
    certificates: HashMap<Hash, Arc<TransactionCertificate>>,
    
    // Deferrals e aborts (sempre presentes)
    deferred: Vec<TransactionDefer>,
    aborted: Vec<TransactionAbort>,
}

// Verificar se bloco está completo
pub fn is_complete(&self) -> bool {
    self.retry_transactions.len() == self.retry_hashes.len()
        && self.priority_transactions.len() == self.priority_hashes.len()
        && self.transactions.len() == self.tx_hashes.len()
        && self.certificates.len() == self.cert_hashes.len()
}
```

## VoteSet (Agregação de Votos)

```rust
pub struct VoteSet {
    // Votos verificados (passaram em verificação de assinatura)
    verified_votes: Vec<(usize, BlockVote, u64)>,
    verified_power: u64,
    
    // Votos não verificados (buffered para batch verification)
    unverified_votes: Vec<(usize, BlockVote, PublicKey, u64)>,
    unverified_power: u64,
    
    // Deduplicação
    seen_validators: Vec<bool>,
    
    // Estado
    pending_verification: bool,
    qc_built: bool,
}

// Otimização: Deferred verification
// Votos não são verificados quando recebidos
// Quando temos quorum, batch verify todos
// Evita verificar votos que nunca usaremos (view change)
```

## CommitmentProof (Cross-Shard)

```rust
pub struct CommitmentProof {
    // Qual transação foi commitada
    pub tx_hash: Hash,
    
    // Em qual shard
    pub source_shard: ShardGroupId,
    
    // Quem assinou (bitfield)
    pub signers: SignerBitfield,
    
    // Assinatura agregada
    pub aggregated_signature: Bls12381G2Signature,
    
    // Em qual altura
    pub block_height: BlockHeight,
    pub block_timestamp: u64,
    
    // Estado que foi commitado (para cycle detection)
    pub entries: Arc<Vec<StateEntry>>,
}

// Tamanho: ~48 bytes (bitfield) + 48 bytes (signature) + entries
// Sem agregação: 67 × 64 bytes = ~4KB
// Com agregação: ~100 bytes + entries
```

---

# Parte 7: Fluxo Completo de Consenso (Exemplo Prático)

## Cenário: 4 Validadores, 1 Shard

```
Validators: V0, V1, V2, V3
Quorum: 3 (2f+1, onde f=1)
Proposer rotation: (height + round) % 4
```

### Height 1, Round 0

**Proposer**: V0 (1 + 0) % 4 = 0

```
T=0ms: V0 on_proposal_timer()
  → BuildProposal action
  → Proposer computa state_root especulativamente
  → Cria BlockHeader
  → Assina header com BLS
  → Broadcast BlockHeaderGossip

T=10ms: V1, V2, V3 recebem header
  → on_block_header()
  → Valida header
  → Aguarda dados (transactions/certificates)
  → Dados chegam via gossip
  → Bloco completo
  → trigger_qc_verification_or_vote()
  → Verifica parent_qc (BLS signature)
  → Callback: on_qc_verified()
  → Verifica state_root
  → Cria BlockVote (BLS)
  → Envia para V0

T=20ms: V0 recebe votos de V1, V2, V3
  → on_block_vote()
  → Adiciona a VoteSet
  → Verifica: 3 votos = quorum
  → Action: VerifyAndBuildQuorumCertificate
  → Batch verifica 3 assinaturas
  → Callback: on_qc_formed()
  → Agrega assinaturas
  → Cria QuorumCertificate
  → Broadcast QC
  → Atualiza latest_qc

T=30ms: V1, V2, V3 recebem QC
  → on_qc_formed()
  → Bloco em height 0 está COMMITADO (two-chain rule)
  → Commit block
  → Atualiza JMT
  → Avança para height 1

T=100ms: Proposal timer para height 1
  → V1 é proposer (1 + 0) % 4 = 1
  → Repeat...
```

### Height 2, Round 0 → Round 1 (View Change)

**Proposer**: V2 (2 + 0) % 4 = 2

```
T=100ms: V2 on_proposal_timer()
  → BuildProposal
  → Broadcast header

T=110ms: V0, V1, V3 recebem header
  → Validam
  → Aguardam dados
  → Timeout: dados não chegam (network partition)

T=200ms: Proposal timer (V0)
  → Timeout! V2 não produziu bloco em tempo
  → Avança view: view = 1
  → Próximo proposer = (2 + 1) % 4 = 3
  → Propõe fallback block (empty)

T=210ms: V1, V2, V3 recebem fallback
  → Votam
  → QC forma
  → Block commitado

T=300ms: Proposal timer (V1)
  → V0 é proposer (2 + 2) % 4 = 0
  → Propõe normal block
  → Quorum formado
  → Consenso avança
```

---

# Parte 8: Debugging e Troubleshooting

## Logs Importantes

```rust
// BFT consensus
info!("Requesting block build for proposal");
info!("Received block header");
debug!("Vote locking: already voted for different block");
info!("Updated latest_qc from received block header");
info!("Bloco commitado via two-chain rule");

// Execution
info!("Executing transaction");
warn!("Transaction execution failed");

// Sync
info!("Missing parent block, triggering sync");
info!("Entering sync mode");
info!("Exiting sync mode");

// View changes
info!("View change: advancing round due to timeout");
info!("Re-proposing vote-locked block after view change");
```

## Métricas Importantes

```rust
pub struct BftStats {
    pub view_changes: u64,           // Quantos view changes?
    pub current_round: u64,          // Em qual round estamos?
    pub committed_height: u64,       // Qual altura commitada?
}

pub struct MempoolStats {
    pub total_transactions: usize,
    pub lock_contention: usize,      // Quantas transações em conflito?
}

pub struct NetworkStats {
    pub messages_sent: u64,
    pub messages_received: u64,
    pub bytes_sent: u64,
    pub bytes_received: u64,
}
```

## Testes Importantes

```rust
// Determinism test
#[test]
fn test_deterministic_simulation() {
    let mut runner1 = SimulationRunner::new(config, seed=42);
    let mut runner2 = SimulationRunner::new(config, seed=42);
    
    runner1.run_until(Duration::from_secs(10));
    runner2.run_until(Duration::from_secs(10));
    
    assert_eq!(runner1.committed_height(), runner2.committed_height());
}

// Safety test
#[test]
fn test_no_conflicting_commits() {
    // Verificar que nunca dois blocos diferentes são commitados na mesma altura
}

// Liveness test
#[test]
fn test_consensus_advances_under_latency() {
    // Verificar que consenso avança mesmo com latência
}
```

