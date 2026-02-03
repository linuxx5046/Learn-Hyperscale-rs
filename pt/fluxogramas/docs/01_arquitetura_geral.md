# Fluxograma 1: Arquitetura Geral do Hyperscale-RS

## Visão Geral

Este fluxograma apresenta a **visão macro do sistema Hyperscale-RS**, mostrando como todos os componentes se conectam e interagem. É o ponto de partida ideal para entender a estrutura geral do projeto.

## Componentes Principais

### Camada de Rede
- **Rede P2P:** Sistema de comunicação entre nós baseado em peer-to-peer
- Responsável por disseminar eventos, blocos e votos entre todos os nós

### Camada de Nó (NodeStateMachine)
- **Orquestrador Central:** Coordena todos os componentes do nó
- Recebe eventos da rede e os roteia para os componentes apropriados
- Gerencia o ciclo de vida de cada componente

### Componentes Principais

#### BFT State Machine
- **Protocolo:** HotStuff-2 (variação de HotStuff)
- **Função:** Implementar consenso Byzantine Fault Tolerant
- **Responsabilidades:**
  - Gerenciar época (epoch) e round
  - Coordenar votação
  - Criar Quorum Certificates (QC)
  - Implementar Two-Chain Rule para commit

#### Execution Engine
- **Função:** Executar transações determinísticamente
- **Responsabilidades:**
  - Executar TX na ordem proposta
  - Atualizar estado (account state)
  - Gerar State Root (hash do estado)
  - Garantir determinismo

#### Mempool
- **Função:** Gerenciar pool de transações pendentes
- **Responsabilidades:**
  - Receber TX da rede
  - Validar TX (assinatura, nonce, saldo)
  - Detectar conflitos
  - Manter fila ordenada

#### Provisioning
- **Função:** Preparar blocos para proposição
- **Responsabilidades:**
  - Selecionar TX do mempool
  - Montar blocos
  - Calcular Merkle Root
  - Preparar para consenso

#### Livelock Prevention
- **Função:** Detectar e prevenir ciclos de execução
- **Responsabilidades:**
  - Detectar ciclos (TX A → B → A)
  - Defer TX para próximo bloco
  - Retry quando ciclo é resolvido
  - Garantir progresso (liveness)

### Camada de Dados

#### Types & Structures
- **Hash:** Identificadores únicos (Blake3)
- **Block:** Estrutura de bloco com TX, Merkle Root, State Root
- **Vote:** Voto de um validador
- **QC:** Quorum Certificate (prova de consenso)

#### Cryptography
- **BLS12-381:** Assinaturas agregáveis (para QC)
- **Ed25519:** Assinaturas rápidas (para votos)
- **Blake3:** Hashing criptográfico

#### Storage
- **State Root:** Hash do estado após execução
- **Ledger:** Registro imutável de blocos
- **Persistência:** Armazenamento durável

### Camada de Produção
- **Runtime:** Executor de threads especializadas
- **Thread Pool Specialization:** Threads dedicadas para diferentes tarefas
- Otimização de performance em produção

## Fluxo de Dados

```
Rede P2P
    ↓
NodeStateMachine (Dispatcher)
    ↓
    ├─→ BFT State Machine
    │       ├─→ Usa: Types, Cryptography
    │       └─→ Gera: QC, Votos
    │
    ├─→ Execution Engine
    │       ├─→ Usa: Types, Cryptography
    │       └─→ Gera: State Root
    │
    ├─→ Mempool
    │       ├─→ Usa: Types, Cryptography
    │       └─→ Gera: Fila de TX validadas
    │
    ├─→ Provisioning
    │       ├─→ Usa: Mempool, Types
    │       └─→ Gera: Blocos preparados
    │
    └─→ Livelock Prevention
            ├─→ Usa: Execution Engine
            └─→ Gera: Retry logic
    
    ↓
Storage (State Root, Ledger)
    ↓
Runtime (Thread Pool)
```

## Conceitos-Chave

### NodeStateMachine
O coração do sistema. Coordena todos os componentes através de um **event dispatcher**:

1. **Evento chega** (rede, timer, TX)
2. **Dispatcher roteia** para componente apropriado
3. **Componente processa** evento
4. **Gera ações** (Vote, Propose, Execute, Broadcast)
5. **Ações atualizam** estado local
6. **Feedback** para dispatcher

### Separação de Responsabilidades
- **BFT:** Apenas consenso
- **Execution:** Apenas execução
- **Mempool:** Apenas gerenciamento de TX
- **Provisioning:** Apenas preparação de blocos
- **Livelock:** Apenas prevenção de ciclos

### Criptografia Modular
- **BLS12-381:** Agregação de assinaturas (eficiente para QC)
- **Ed25519:** Assinaturas individuais (rápidas)
- **Blake3:** Hashing (rápido e seguro)

## Quando Usar Este Fluxograma

✅ **Use quando:**
- Precisa entender a estrutura geral do sistema
- Quer saber como os componentes se conectam
- Está começando a aprender Hyperscale-RS
- Precisa explicar o sistema para outros

❌ **Não use quando:**
- Precisa entender detalhes de consenso (veja #02)
- Precisa entender fluxo de transações (veja #03)
- Precisa entender votação (veja #05)

## Próximos Passos

Após entender este fluxograma, recomenda-se:

1. **Máquina de Estado do Nó (#04)** - Entender como NodeStateMachine funciona internamente
2. **Ciclo de Consenso BFT (#02)** - Entender como BFT State Machine funciona
3. **Fluxo de Transações (#03)** - Entender como TX são processadas

## Referências

- **Arquivo Mermaid:** `mermaid/01_arquitetura_geral.mmd`
- **Imagem:** `images/01_arquitetura_geral.png`
- **Repositório:** https://github.com/flightofthefox/hyperscale-rs

---

**Última atualização:** 03 de Fevereiro de 2026
