# Análisis de Arquitectura Hyperscale-RS

## Visión General del Proyecto

**Hyperscale-rs** es una implementación en Rust de un protocolo de consenso distribuido de alto rendimiento basado en **HotStuff-2** con optimizaciones para sistemas de fragmentación (state sharding).

### Estadísticas del Proyecto
- **Líneas de código**: ~76,477 líneas de Rust
- **Número de crates**: 16 crates especializados
- **Arquitectura**: Modular, basada en máquinas de estado síncronas
- **Enfoque**: Determinismo, testabilidad, rendimiento en producción

---

## Estructura de Crates (Dependencias Jerárquicas)

### Capa 1: Tipos Fundamentales
```
hyperscale-types (6,988 líneas)
├─ Primitivos: Hash, Keys (Ed25519, BLS12-381)
├─ Identificadores: ValidatorId, BlockHeight, ShardGroupId
├─ Consenso: Block, BlockHeader, QuorumCertificate
├─ Transacciones: RoutableTransaction, TransactionCertificate
└─ Topología: Conjuntos de validadores, configuración de shards
```

**Características Clave:**
- Sin dependencias de otros crates del workspace
- Usa **BLS12-381** para firmas agregables (QCs)
- Usa **Ed25519** para firmas rápidas (transacciones)
- Árboles de Merkle con hojas etiquetadas para pruebas de posición

### Capa 2: Abstracciones Centrales
```
hyperscale-core (2,126 líneas)
├─ Trait StateMachine (síncrono, determinístico)
├─ Enum Event (entrada del sistema)
├─ Enum Action (salida del sistema)
└─ Trait StateRootComputer (integración JMT)
```

**Filosofía:**
- **Síncrono**: Sin async/await
- **Determinístico**: Mismo estado + evento = mismas acciones
- **Puro**: Sin I/O (todo devuelto como Actions)

### Capa 3: Máquinas de Estado Especializadas

#### Consenso BFT (10,801 líneas)
```
hyperscale-bft
├─ BftState (máquina de estado principal)
├─ Proposal: propuesta de bloques y broadcasting
├─ Voting: recolección de votos y formación de QC
├─ View Changes: implícitos (HotStuff-2)
└─ Vote Locking: crítico para seguridad
```

**Protocolo HotStuff-2:**
- **Commit de dos cadenas**: Bloque en altura H se confirma cuando se forma QC en H+1
- **Cambios de vista implícitos**: Timeout local, sin coordinación
- **Bloqueo de votos**: Previene equivocación (seguridad)
- **Regla de desbloqueo**: QC en altura H desbloquea votos en ≤H

#### Ejecución (3,464 líneas)
```
hyperscale-execution
├─ ExecutionState (coordinación de ejecución)
├─ Ejecución de shard único
├─ Coordinación entre shards (2PC)
└─ Provisión de estado
```

#### Mempool (3,132 líneas)
```
hyperscale-mempool
├─ Envío de transacciones
├─ Coordinación de gossip
├─ Detección de conflictos
└─ Seguimiento de estado
```

#### Prevención de Livelock (1,294 líneas)
```
hyperscale-livelock
├─ Detección de ciclos (entre shards)
├─ Prevención de deadlock
└─ Coordinación de diferimiento
```

#### Provisiones (1,580 líneas)
```
hyperscale-provisions
├─ Coordinación centralizada de provisiones
├─ Soporte para transacciones entre shards
└─ Consistencia de estado
```

### Capa 4: Composición
```
hyperscale-node (1,109 líneas)
└─ NodeStateMachine
   ├─ BftState
   ├─ ExecutionState
   ├─ MempoolState
   ├─ ProvisionCoordinator
   └─ LivelockState
```

### Capa 5: Simulación y Producción

#### Simulación (4,862 líneas)
```
hyperscale-simulation
├─ SimulationRunner (determinístico)
├─ Event Queue (BTreeMap ordenado)
├─ SimStorage (en memoria)
├─ SimulatedNetwork (latencia configurable)
└─ NetworkTrafficAnalyzer
```

**Características:**
- Completamente determinístico
- Misma semilla = mismos resultados
- Soporta latencia artificial
- Análisis de tráfico de red

#### Producción (21,965 líneas)
```
hyperscale-production
├─ ProductionRunner (async/tokio)
├─ Networking con Libp2p
├─ Almacenamiento RocksDB
├─ Thread pools especializados
│  ├─ Crypto pool (rayon)
│  ├─ Execution pool (rayon)
│  └─ I/O pool (tokio)
├─ SyncManager
├─ ValidationBatcher
└─ Telemetría (Prometheus)
```

**Arquitectura de Threads:**
```
Core 0 (anclado): State Machine + Event Loop
    ↓
    ├→ Crypto Pool (rayon): Verificación BLS
    ├→ Execution Pool (rayon): Radix Engine
    └→ I/O Pool (tokio): Red, Almacenamiento, Timers
```

---

## Flujo de Datos (Dirigido por Eventos)

### Ciclo Principal
```
1. Evento entra a StateMachine
   ↓
2. StateMachine.handle(event) → Vec<Action>
   ↓
3. Runner ejecuta Actions
   ├─ SendMessage → red
   ├─ SetTimer → timers
   ├─ VerifyCrypto → crypto pool
   ├─ ExecuteTransaction → execution pool
   ├─ CommitState → almacenamiento
   └─ ...
   ↓
4. Resultados de Actions → nuevos Events
   ↓
5. De vuelta al paso 1
```

### Ejemplo: Consenso de Bloque

```
ProposalTimer
    ↓
BftState.on_proposal_timer()
    → Action::ProposeBlock
    ↓
Runner transmite header del bloque
    ↓
BlockHeaderReceived (en todos los nodos)
    ↓
BftState.on_block_header()
    → Validar, ensamblar, votar
    → Action::SendBlockVote
    ↓
BlockVoteReceived (agregación en el proposer)
    ↓
BftState.on_block_vote()
    → Recolectar votos, formar QC
    → Action::QuorumCertificateFormed
    ↓
QuorumCertificateFormed
    ↓
BftState.on_qc_formed()
    → Actualizar estado de cadena
    → Confirmar si está listo (two-chain)
    → Action::CommitBlock
```

---

## Componentes Críticos de Seguridad

### 1. Criptografía
- **BLS12-381** (G1 privado, G2 firmas)
  - Firmas agregables (QCs)
  - Verificación en batch
  - Verificación de prueba de ciclo
  
- **Ed25519**
  - Firmas rápidas (transacciones)
  - Verificación en batch
  - Notarización de transacciones

### 2. Consenso (HotStuff-2)
- **Bloqueo de Votos**: Previene equivocación
  - Rastreado en `voted_heights: HashMap<u64, (Hash, u64)>`
  - Persistido en almacenamiento (crítico para seguridad)
  
- **Intersección de Quórum**: 2f+1 de n=3f+1
  - Dos quórums cualesquiera se superponen en ≥1 validador honesto
  - Previene conflictos
  
- **Commit de Dos Cadenas**: Finalidad rápida
  - Bloque H confirmado cuando QC se forma en H+1
  - Garantiza finalidad incluso bajo asincronía

### 3. Ejecución Distribuida
- **2PC entre Shards**: Two-Phase Commit
  - Coordinación vía provisiones
  - Diferimiento para prevenir deadlock
  
- **Provisión de Estado**: Consistencia
  - Raíces de estado JMT (Jellyfish Merkle Tree)
  - Verificación de raíz de estado antes de votar
  - Ejecución especulativa con rollback

### 4. Prevención de Livelock
- **Detección de Ciclos**: Detecta ciclos entre shards
- **Diferimiento**: Difiere transacciones problemáticas
- **Reintento**: Reintenta después de resolución

---

## Patrones de Software de Producción

### 1. Patrón de Máquina de Estado
- Toda la lógica es síncrona y determinística
- Facilita pruebas, simulación, depuración
- Separación clara entre lógica e I/O

### 2. Patrón de Agregador de Eventos
- Una sola tarea posee la máquina de estado
- Eventos vía canal mpsc
- Evita contención de mutex

### 3. Especialización de Thread Pools
- **Crypto pool**: Operaciones criptográficas
- **Execution pool**: Radix Engine (CPU-bound)
- **I/O pool**: Red, almacenamiento, timers
- Cada pool optimizado para su carga de trabajo

### 4. Simulación Determinística
- Misma semilla = mismos resultados
- Facilita depuración de race conditions
- Pruebas reproducibles

### 5. Procesamiento por Lotes
- Verificación en batch de firmas
- Validación en batch de transacciones
- Reduce overhead de cambio de contexto

---

## Flujo Detallado de Consenso

### Fase 1: Propuesta
```
Proposer (round-robin):
  1. Selecciona transacciones del mempool
  2. Ejecuta especulativamente
  3. Calcula raíz de estado (JMT)
  4. Crea BlockHeader
  5. Firma header (BLS)
  6. Transmite BlockHeader
  7. Espera ensamblado de datos
```

### Fase 2: Votación
```
Validador recibe BlockHeader:
  1. Valida firma del proposer
  2. Espera datos completos (gossip/fetch)
  3. Valida estado (raíz de estado)
  4. Valida transacciones
  5. Verifica bloqueo de voto (seguridad)
  6. Crea BlockVote (BLS)
  7. Envía al proposer
```

### Fase 3: Formación de QC
```
Proposer recolecta votos:
  1. Valida cada voto (BLS)
  2. Espera 2f+1 votos
  3. Agrega firmas (agregación BLS)
  4. Crea QuorumCertificate
  5. Transmite QC
  6. Actualiza estado de cadena
```

### Fase 4: Confirmación
```
Cuando QC se forma para altura H+1:
  1. Bloque en altura H es confirmado
  2. Actualizar committed_height
  3. Desbloquear votos ≤H
  4. Ejecutar callbacks de commit
```

---

# Parte 2: Transacciones entre Shards

## Problema

**Atomicidad**: Una transacción puede afectar múltiples shards.

```
TX: Transferir 100 tokens de Alice (Shard 0) a Bob (Shard 1)

Problema:
- Shard 0 debe debitar a Alice
- Shard 1 debe acreditar a Bob
- Ambas operaciones deben ser atómicas
```

## Solución: Two-Phase Commit (2PC)

### Fase 1: Preparar (Source Shard)
```
1. Shard 0 recibe transacción
2. Ejecuta localmente (debita a Alice)
3. Calcula StateProvisions (estado a provisionar)
4. Incluye en bloque
5. Forma QuorumCertificate (commit local)
```

### Fase 2: Commit (Target Shard)
```
1. Shard 1 recibe CommitmentProof
2. Valida que Shard 0 commitió
3. Ejecuta localmente (acredita a Bob)
4. Incluye en bloque
5. Forma QuorumCertificate (commit final)
```

## StateProvision

```rust
pub struct StateProvision {
    pub tx_hash: Hash,
    pub source_shard: ShardGroupId,
    pub target_shard: ShardGroupId,
    
    // Estado provisionado
    pub entries: Vec<StateEntry>,
    
    // Firma BLS del validador
    pub signature: Bls12381G2Signature,
    
    pub block_height: BlockHeight,
    pub block_timestamp: u64,
}

pub struct StateEntry {
    pub address: ComponentAddress,
    pub key: Vec<u8>,
    pub value: Vec<u8>,
}
```

## CommitmentProof

```rust
pub struct CommitmentProof {
    pub tx_hash: Hash,
    pub source_shard: ShardGroupId,
    
    // Quién firmó (bitfield)
    pub signers: SignerBitfield,
    
    // Firma agregada
    pub aggregated_signature: Bls12381G2Signature,
    
    pub block_height: BlockHeight,
    pub block_timestamp: u64,
    
    // Estado (único, compartido)
    pub entries: Arc<Vec<StateEntry>>,
}
```

## Flujo Completo

```
TX submitted to Shard 0
    ↓
Shard 0: Execute → StateProvisions
    ↓
Include in Block @ H
    ↓
Form QC @ H
    ↓
Validators sign StateProvisions (BLS)
    ↓
Aggregate into CommitmentProof
    ↓
Send CommitmentProof to Shard 1
    ↓
Shard 1: Validate CommitmentProof
    ↓
Include in Block @ H'
    ↓
Execute using provisioned state
    ↓
Form QC @ H'
    ↓
TX committed on both shards
```

---

# Parte 3: Componentes Criptográficos

## BLS12-381 (Firmas Agregables)

**Propiedades:**
- Firma: G2 (96 bytes)
- Clave pública: G1 (48 bytes)
- Agregable: múltiples firmas → una firma
- Verificación en batch

**Uso en Hyperscale:**

### 1. QuorumCertificates
```rust
pub struct QuorumCertificate {
    pub block_hash: Hash,
    pub block_height: BlockHeight,
    
    // Quién firmó (bitfield compacto)
    pub signers: SignerBitfield,  // 1 bit por validador
    
    // Firma agregada (96 bytes total)
    pub aggregated_signature: Bls12381G2Signature,
}

// Verificación:
// 1. Reconstruir mensaje firmado
let message = signing_domain("QuorumCertificate") 
    + block_hash 
    + block_height;

// 2. Agregar claves públicas
let aggregated_pubkey = signers.iter()
    .map(|i| validator_set[i].pubkey)
    .aggregate();

// 3. Verificar firma agregada
verify(aggregated_signature, aggregated_pubkey, message)?;
```

### 2. StateProvisions
```rust
// Cada validador firma provision
let signature = bls_sign(
    domain: "StateProvision",
    message: provision.to_bytes(),
    secret_key: validator.secret_key,
);

// Agregar cuando tenemos quórum
let aggregated_signature = signatures.iter()
    .aggregate();

// Crear CommitmentProof
CommitmentProof {
    signers,
    aggregated_signature,
    ...
}
```

### 3. Cycle Proofs
```rust
// Prueba de que se detectó un ciclo
pub struct CycleProof {
    pub winner_tx_hash: Hash,
    pub loser_tx_hash: Hash,
    
    // Proof que winner fue committed
    pub winner_commitment: CommitmentProof,
    
    // Firmado por quórum de source shard
    pub aggregated_signature: Bls12381G2Signature,
}
```

## Ed25519 (Firmas Rápidas)

**Propiedades:**
- Firma: 64 bytes
- Clave pública: 32 bytes
- Rápido: ~50k firmas/seg
- No agregable

**Uso en Hyperscale:**

### 1. Transacciones de Usuario
```rust
pub struct Transaction {
    pub from: PublicKey,
    pub to: Address,
    pub amount: u64,
    pub nonce: u64,
    
    // Firma Ed25519
    pub signature: Ed25519Signature,
}

// Verificación
verify_ed25519(
    transaction.signature,
    transaction.from,
    transaction.to_bytes(),
)?;
```

### 2. Notarización de Transacciones
```rust
// Cada validador que ve la transacción la firma
let notarization = ed25519_sign(
    message: tx.hash(),
    secret_key: validator.secret_key,
);

// Se recolectan múltiples notarizaciones
pub struct TransactionCertificate {
    pub tx_hash: Hash,
    pub notarizations: Vec<(ValidatorId, Ed25519Signature)>,
}
```

## Dominios de Firma (Separación de Contexto)

```rust
pub enum SigningDomain {
    QuorumCertificate,
    BlockVote,
    StateProvision,
    CycleProof,
    TransactionNotarization,
}

fn signing_domain(domain: &str) -> Hash {
    // Prefijo único para cada dominio
    hash(format!("HYPERSCALE_V1_{}", domain))
}

// Uso:
let message = signing_domain("BlockVote") 
    + block_hash 
    + block_height;
```

**Propósito**: Prevenir ataques de replay entre contextos.

---

# Parte 4: Coordinación de Provisiones

## Problema

**Estado Fragmentado**: Cada shard mantiene un subconjunto del estado global.

```
Shard 0: Cuentas A-M
Shard 1: Cuentas N-Z

TX: A → B  (mismo shard) ✓
TX: A → N  (entre shards) ✗ Shard 1 no tiene estado de A
```

## Solución: State Provisioning

### Concepto

Cuando una transacción afecta múltiples shards:
1. **Source shard** ejecuta y genera StateProvisions
2. **Target shard** recibe provisiones y ejecuta

### Ejemplo Detallado

```rust
// TX en Shard 0
let tx = Transaction {
    from: Alice,  // en Shard 0
    to: Bob,      // en Shard 1
    amount: 100,
};

// Shard 0 ejecuta
let result = execute(tx);
// → Alice.balance -= 100

// Shard 0 genera provision
let provision = StateProvision {
    tx_hash: hash(tx),
    source_shard: 0,
    target_shard: 1,
    entries: vec![
        StateEntry {
            address: Alice.address,
            key: "balance",
            value: encode(900),  // nuevo balance
        },
        StateEntry {
            address: tx,
            key: "metadata",
            value: encode(metadata),
        },
    ],
    signature: sign_bls(provision),
};
```

### Agregación en CommitmentProof

```rust
// Proposer recolecta provisions de validators
let provisions: Vec<StateProvision> = collect_from_validators();

// Verificar que todos coinciden
assert!(all_provisions_match(provisions));

// Agregar firmas
let aggregated_signature = provisions.iter()
    .map(|p| p.signature)
    .aggregate();

// Crear proof
let proof = CommitmentProof {
    tx_hash,
    source_shard: 0,
    signers: extract_signers(provisions),
    aggregated_signature,
    entries: Arc::new(provisions[0].entries.clone()),
};

// Enviar a Shard 1
send_to_shard(1, proof);
```

### Ejecución en Target Shard

```rust
// Shard 1 recibe CommitmentProof
fn on_commitment_proof(proof: CommitmentProof) {
    // 1. Validar firma agregada
    validate_commitment_proof(&proof)?;
    
    // 2. Incluir en bloque
    block.commitment_proofs.push(proof.clone());
    
    // 3. Ejecutar transacción usando estado provisionado
    execute_with_provisions(proof.tx_hash, proof.entries);
    // → Bob.balance += 100
}
```

## ProvisionCoordinator (Centralizado)

```rust
pub struct ProvisionCoordinator {
    // Pending provisions by tx_hash
    pending: HashMap<Hash, PendingProvision>,
    
    // Completed proofs by tx_hash
    completed: HashMap<Hash, CommitmentProof>,
}

pub struct PendingProvision {
    tx_hash: Hash,
    target_shard: ShardGroupId,
    
    // Provisions from validators
    provisions: Vec<(ValidatorId, StateProvision)>,
    
    // Track which validators signed
    signers: HashSet<ValidatorId>,
}

impl ProvisionCoordinator {
    pub fn on_state_provision(
        &mut self,
        validator_id: ValidatorId,
        provision: StateProvision,
    ) -> Option<Action> {
        let pending = self.pending
            .entry(provision.tx_hash)
            .or_insert_with(|| PendingProvision::new(provision.tx_hash));
        
        // Add provision
        pending.provisions.push((validator_id, provision));
        pending.signers.insert(validator_id);
        
        // Check if we have quorum
        if has_quorum(&pending.signers) {
            // Aggregate into CommitmentProof
            let proof = self.aggregate_proof(pending)?;
            self.completed.insert(proof.tx_hash, proof.clone());
            
            // Send to target shard
            return Some(Action::SendCommitmentProof {
                target_shard: pending.target_shard,
                proof,
            });
        }
        
        None
    }
}
```

## Flujo de Provisión Completo

### Fase 1: Prepare (Source Shard)
```
1. Transaction ejecutada en source shard
2. Genera StateProvisions
3. Cada validador firma provision con BLS (dominio StateProvision)
4. Envía a target shard
```

### Fase 2: Commit (Target Shard)
```
1. Target shard recibe StateProvisions
2. Valida firmas (verificación en batch)
3. Agrega en CommitmentProof (agregación BLS)
4. Incluye CommitmentProof en bloque
5. Transacción puede ejecutarse en target shard
```

## Prevención de Livelock

**Problema**: Los ciclos entre shards causan deadlock.

```
TX A: Shard 0 → Shard 1
TX B: Shard 1 → Shard 0

Shard 0 espera provisión de B (que está en Shard 1)
Shard 1 espera provisión de A (que está en Shard 0)
¡DEADLOCK!
```

**Solución**: Detección de Ciclos + Diferimiento

```rust
// Detectar ciclo
fn detect_cycle(tx: &Transaction) -> Option<CycleProof> {
    // Verificar si hay un camino de regreso
    // Si sí, generar CycleProof (firmado por quórum)
}

// Diferir transacción
fn defer_transaction(tx: &Transaction, proof: CycleProof) {
    // Incluir en bloque como TransactionDefer
    // Devolver con nuevo hash (reintentar)
    // Intentar nuevamente en bloque futuro
}
```

**CycleProof**: Prueba de que el ciclo fue detectado por quórum de validadores.

```rust
pub struct CycleProof {
    // Qué transacción ganó (será ejecutada)
    pub winner_tx_hash: Hash,
    
    // Qué transacción perdió (será diferida)
    pub loser_tx_hash: Hash,
    
    // CommitmentProof del ganador (prueba de que fue committed)
    pub winner_commitment: CommitmentProof,
    
    // Firmado por quórum de source shard
    pub aggregated_signature: Bls12381G2Signature,
}
```

---

# Parte 5: Patrones de Software de Producción

## Patrón de Máquina de Estado

**Concepto**: Toda la lógica es síncrona, determinística, sin I/O.

```rust
pub trait StateMachine {
    fn handle(&mut self, event: Event) -> Vec<Action>;
    fn set_time(&mut self, now: Duration);
    fn now(&self) -> Duration;
}

// Implementación
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

**Beneficios:**
- Testeable (sin dependencias externas)
- Determinístico (mismo estado + evento = mismas acciones)
- Simulable (corre en simulación determinística)
- Depurable (traza completa de eventos)

## Patrón de Agregador de Eventos (Producción)

```rust
// Una sola tarea posee la máquina de estado
async fn run_state_machine(
    mut state_machine: NodeStateMachine,
    mut event_rx: mpsc::Receiver<Event>,
) {
    loop {
        // Recibir evento
        let event = event_rx.recv().await;
        
        // Procesar (síncrono)
        let actions = state_machine.handle(event);
        
        // Ejecutar acciones (I/O)
        for action in actions {
            execute_action(action).await;
        }
    }
}

// Múltiples productores de eventos
// Red → canal mpsc → Agregador de Eventos
// Timers → canal mpsc → Agregador de Eventos
// Almacenamiento → canal mpsc → Agregador de Eventos
```

**Beneficios:**
- Sin mutex (un solo propietario)
- Sin contención (procesamiento serial)
- Sin race conditions (eventos ordenados)

## Especialización de Thread Pool (Producción)

```rust
pub struct ThreadPoolManager {
    // Crypto pool: Verificación BLS, chequeos de firma
    crypto_pool: rayon::ThreadPool,
    
    // Execution pool: Radix Engine, cómputo de merkle
    execution_pool: rayon::ThreadPool,
    
    // I/O pool: runtime tokio para red/almacenamiento/timers
    io_runtime: tokio::runtime::Runtime,
}

// Despachar acciones al pool apropiado
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

**Configuración de Threads**:
```rust
// Auto-detectar cores
let config = ThreadPoolConfig::auto();
// 25% crypto, 50% ejecución, 25% I/O

// O personalizar
let config = ThreadPoolConfig::builder()
    .crypto_threads(4)
    .execution_threads(8)
    .io_threads(2)
    .pin_cores(true)  // Solo Linux
    .build()?;
```

## Procesamiento por Lotes

### Verificación en Batch de Firmas
```rust
// En lugar de verificar cada voto individualmente:
for vote in votes {
    verify_signature(&vote)?;  // Lento
}

// Verificar en batch cuando tenemos quórum:
Action::VerifyAndBuildQuorumCertificate {
    votes,
    public_keys,
    // Runner verifica todos en paralelo
}
```

### Validación en Batch de Transacciones
```rust
// Validar múltiples transacciones en paralelo
let validation_results = validation_pool.par_iter(transactions)
    .map(|tx| validate_transaction(tx))
    .collect();
```

## Simulación Determinística

```rust
pub struct SimulationRunner {
    // Cola de eventos ordenada por: (tiempo, prioridad, nodo, secuencia)
    event_queue: BTreeMap<EventKey, Event>,
    
    // Nodos (en proceso)
    nodes: Vec<NodeStateMachine>,
    
    // Almacenamiento (en memoria)
    storage: SimStorage,
    
    // Red (latencia simulada)
    network: SimulatedNetwork,
}

// Determinístico: misma semilla = mismos resultados
let mut runner = SimulationRunner::new(config, seed=42);
runner.initialize_genesis();
runner.run_until(Duration::from_secs(10));

// Traza completa de eventos
for event in runner.event_log() {
    println!("{:?}", event);
}
```

**Usos:**
- Pruebas de consenso
- Depuración de race conditions
- Verificación de liveness
- Análisis de rendimiento

---

# Parte 6: Estructuras de Datos Principales

## PendingBlock (Ensamblado de Bloques)

```rust
pub struct PendingBlock {
    // Header (siempre presente)
    header: BlockHeader,
    
    // Hashes de transacciones (siempre presentes)
    retry_hashes: Vec<Hash>,
    priority_hashes: Vec<Hash>,
    tx_hashes: Vec<Hash>,
    
    // Datos reales (pueden faltar)
    retry_transactions: Option<Vec<Arc<RoutableTransaction>>>,
    priority_transactions: Option<Vec<Arc<RoutableTransaction>>>,
    transactions: Option<Vec<Arc<RoutableTransaction>>>,
    
    // Certificados (pueden faltar)
    certificates: Option<Vec<Arc<TransactionCertificate>>>,
    
    // Seguimiento de estado
    is_complete: bool,
    verification_status: VerificationStatus,
}
```

**Estados:**
- **Assembling**: Header recibido, esperando datos
- **Complete**: Todos los datos recibidos
- **Verified**: Todas las verificaciones pasadas
- **Voted**: Voto enviado

## VoteSet (Agregación de Votos)

```rust
pub struct VoteSet {
    verified_votes: Vec<(usize, BlockVote, u64)>,
    unverified_votes: Vec<(usize, BlockVote, PublicKey, u64)>,
    pending_verification: bool,
}

impl VoteSet {
    pub fn try_build_qc(&mut self) -> Option<Action> {
        // Si tenemos quórum de votos no verificados
        if self.unverified_power >= quorum_threshold {
            // Verificar todos en batch
            return Some(Action::VerifyAndBuildQuorumCertificate {
                votes: self.unverified_votes.clone(),
                public_keys: self.collect_public_keys(),
            });
        }
        None
    }
}
```

## CommitmentProof (Agregación entre Shards)

```rust
pub struct CommitmentProof {
    pub tx_hash: Hash,
    pub source_shard: ShardGroupId,
    
    // Quién firmó (bitfield)
    pub signers: SignerBitfield,
    
    // Firma agregada
    pub aggregated_signature: Bls12381G2Signature,
    
    pub block_height: BlockHeight,
    pub block_timestamp: u64,
    
    // Estado (único, compartido)
    pub entries: Arc<Vec<StateEntry>>,
}
```

---

## Conclusión

Este análisis técnico proporciona una visión integral de la arquitectura de Hyperscale-RS. El sistema demuestra cómo múltiples técnicas sofisticadas—desde primitivos criptográficos hasta protocolos de consenso y patrones de software de producción—trabajan juntas para crear un sistema distribuido de alto rendimiento, tolerante a fallas bizantinas, capaz de manejar transacciones complejas entre shards mientras mantiene garantías de seguridad, vivacidad y rendimiento.

El diseño modular, con clara separación entre la lógica de máquina de estado y las operaciones de I/O, hace que el sistema sea tanto testeable como mantenible. El uso de simulación determinística permite pruebas exhaustivas de escenarios de fallo complejos, mientras que la arquitectura de producción con thread pools especializados asegura una utilización eficiente de recursos y alto rendimiento.
