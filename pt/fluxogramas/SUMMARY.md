# Sumário de Fluxogramas - Hyperscale-RS

## Visão Geral Estruturada

| # | Nome | Objetivo | Componentes Principais | Complexidade | Tempo de Leitura |
|---|------|---------|----------------------|--------------|-----------------|
| 01 | Arquitetura Geral | Visão macro do sistema | Camadas: Rede, Nó, Componentes, Dados, Produção | ⭐ Iniciante | 5 min |
| 02 | Ciclo de Consenso BFT | Fluxo completo de consenso | Época, Round, Líder, Proposição, Votação, QC, Execução | ⭐⭐ Intermediário | 15 min |
| 03 | Fluxo de Transações | Caminho da TX até finalização | Recepção, Validação, Mempool, Proposição, Consenso, Execução | ⭐⭐ Intermediário | 10 min |
| 04 | Máquina de Estado do Nó | Integração de componentes | BFT State, Execution State, Mempool, Provisioning, Livelock | ⭐⭐ Intermediário | 12 min |
| 05 | Ciclo de Votação e QC | Processo de votação detalhado | Vote Locking, Broadcast, Coleta, Quórum, Agregação BLS | ⭐⭐⭐ Avançado | 20 min |
| 06 | Execução Distribuída | Transações cross-shard | Intra-Shard, Cross-Shard, Prepare-Commit, Livelock Prevention | ⭐⭐⭐ Avançado | 18 min |
| 07 | Ciclo Completo de uma Época | Exemplo prático com validadores | 4 Validadores, múltiplos Rounds, Two-Chain Rule, Commit | ⭐⭐ Intermediário | 12 min |
| 08 | Tratamento de Falhas e View Change | Recuperação de falhas de líder | Timeout, Unlock, View Change, Novo Líder, Sincronização | ⭐⭐⭐ Avançado | 15 min |

## Fluxo de Aprendizado Recomendado

### Caminho Iniciante (45 minutos)
```
01. Arquitetura Geral (5 min)
    ↓
02. Ciclo de Consenso BFT (15 min)
    ↓
03. Fluxo de Transações (10 min)
    ↓
07. Ciclo Completo de uma Época (12 min)
```

### Caminho Intermediário (90 minutos)
```
01. Arquitetura Geral (5 min)
    ↓
04. Máquina de Estado do Nó (12 min)
    ↓
02. Ciclo de Consenso BFT (15 min)
    ↓
03. Fluxo de Transações (10 min)
    ↓
07. Ciclo Completo de uma Época (12 min)
    ↓
08. Tratamento de Falhas (15 min)
    ↓
05. Ciclo de Votação e QC (20 min)
```

### Caminho Avançado (120+ minutos)
```
Todos os fluxogramas em ordem:
01 → 04 → 02 → 03 → 07 → 05 → 08 → 06
(com leitura profunda do código-fonte)
```

## Conceitos por Fluxograma

### 01 - Arquitetura Geral
**Conceitos:** Camadas, Componentes, Orquestração
- NodeStateMachine (orquestrador)
- BFT State Machine
- Execution Engine
- Mempool
- Provisioning
- Livelock Prevention
- Criptografia (BLS12-381, Ed25519, Blake3)

### 02 - Ciclo de Consenso BFT
**Conceitos:** Época, Round, Consenso, Execução
- Época (Epoch)
- Round
- Eleição de Líder
- Proposição de Bloco
- Votação (2f+1)
- Quorum Certificate (QC)
- Two-Chain Rule
- Execução Determinística

### 03 - Fluxo de Transações
**Conceitos:** Pipeline de TX, Mempool, Finalização
- Recepção de TX
- Validação (assinatura, nonce, saldo)
- Mempool (fila, detecção de conflitos)
- Proposição (seleção de TX)
- Consenso (votação, QC)
- Execução (determinística)
- Finalização (imutabilidade)

### 04 - Máquina de Estado do Nó
**Conceitos:** Integração, Dispatcher, Event Loop
- BFT State (Epoch, Round, View)
- Execution State (Account State)
- Mempool State (TX pendentes)
- Provisioning State (Blocos)
- Livelock State (Cycle Detection)
- Event Dispatcher
- Action Generation

### 05 - Ciclo de Votação e QC
**Conceitos:** Segurança, Agregação, Verificação
- Vote Locking (safety)
- Unlock Rule (liveness)
- Assinatura Ed25519
- Batch Verification
- Agregação BLS12-381
- Quorum Certificate
- Validação QC

### 06 - Execução Distribuída
**Conceitos:** Sharding, Cross-Shard, Deadlock Prevention
- Intra-Shard (execução direta)
- Cross-Shard (prepare-commit)
- CommitmentProof
- Livelock Prevention
- Cycle Detection
- Retry Logic

### 07 - Ciclo Completo de uma Época
**Conceitos:** Exemplo Prático, Múltiplos Rounds
- 4 Validadores
- Tolerância a falhas (f=1)
- Quórum (2f+1 = 3)
- Múltiplos Rounds
- Two-Chain Rule
- Commit

### 08 - Tratamento de Falhas e View Change
**Conceitos:** Resiliência, Recuperação, Liveness
- Timeout Detection
- Vote Unlock
- View Change
- Novo Líder
- Sincronização
- HighQC
- Recuperação

## Mapa de Dependências

```
Nenhuma dependência
    ↓
01. Arquitetura Geral
    ↓
    ├─→ 04. Máquina de Estado (requer 01)
    │       ↓
    │       ├─→ 02. Consenso BFT (requer 01, 04)
    │       │       ↓
    │       │       ├─→ 05. Votação e QC (requer 02)
    │       │       └─→ 08. Falhas (requer 02, 05)
    │       │
    │       ├─→ 03. Fluxo TX (requer 01, 02)
    │       │       ↓
    │       │       └─→ 06. Execução Distribuída (requer 03, 04)
    │       │
    │       └─→ 07. Ciclo Completo (requer 02)
    │
    └─→ Todos os fluxogramas podem ser consultados após 01
```

## Guia de Uso por Perfil

### Desenvolvedor Novo no Projeto
1. Leia 01 (Arquitetura)
2. Leia 02 (Consenso)
3. Leia 03 (Transações)
4. Consulte 05 e 08 conforme necessário

### Pesquisador / Acadêmico
1. Leia todos em ordem
2. Foque em 02, 05, 08 (segurança e resiliência)
3. Estude 06 (execução distribuída)

### Contribuidor de Código
1. Leia 01 (entender arquitetura)
2. Leia 04 (entender integração)
3. Leia fluxogramas específicos do componente

### Auditor / Revisor de Segurança
1. Leia 02 (consenso)
2. Leia 05 (votação e segurança)
3. Leia 08 (resiliência)
4. Leia 06 (execução distribuída)

## Estatísticas

- **Total de Fluxogramas:** 8
- **Tempo Total de Leitura:** ~100 minutos
- **Nível Iniciante:** 1 fluxograma (5 min)
- **Nível Intermediário:** 4 fluxogramas (47 min)
- **Nível Avançado:** 3 fluxogramas (53 min)
- **Formatos Disponíveis:** Mermaid (.mmd), PNG (.png), Documentação (.md)

---

**Última atualização:** 03 de Fevereiro de 2026
