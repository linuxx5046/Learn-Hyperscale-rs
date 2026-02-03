# Fluxogramas do Hyperscale-RS

Conjunto completo de fluxogramas modulares que detalham o funcionamento do software Hyperscale-RS, com ênfase no ciclo de consenso, época, block, round e componentes principais.

## Conteúdo

Os fluxogramas estão organizados em três formatos complementares:

- **`mermaid/`** - Código-fonte dos diagramas (`.mmd`) - formato de edição
- **`images/`** - Imagens PNG renderizadas - para visualização rápida
- **`docs/`** - Documentação detalhada de cada fluxograma

## Índice de Fluxogramas

| # | Nome | Descrição | Nível | Pré-requisitos |
|---|------|-----------|-------|----------------|
| 01 | [Arquitetura Geral](docs/01_arquitetura_geral.md) | Visão macro do sistema e seus componentes principais | Iniciante | Nenhum |
| 02 | [Ciclo de Consenso BFT](docs/02_ciclo_consenso_bft.md) | Fluxo completo de consenso, desde eleição do líder até execução | Intermediário | #01 |
| 03 | [Fluxo de Transações](docs/03_fluxo_transacoes.md) | Caminho de uma transação desde recepção até finalização | Intermediário | #01, #02 |
| 04 | [Máquina de Estado do Nó](docs/04_node_state_machine.md) | Como o NodeStateMachine integra todos os componentes | Intermediário | #01 |
| 05 | [Ciclo de Votação e QC](docs/05_ciclo_votacao_qc.md) | Processo detalhado de votação e criação de Quorum Certificate | Avançado | #02 |
| 06 | [Execução Distribuída](docs/06_execucao_distribuida.md) | Como transações cross-shard são executadas | Avançado | #03, #04 |
| 07 | [Ciclo Completo de uma Época](docs/07_ciclo_completo_epoca.md) | Exemplo prático com 4 validadores em múltiplos rounds | Intermediário | #02 |
| 08 | [Tratamento de Falhas e View Change](docs/08_view_change_falhas.md) | Como o sistema se recupera de falhas de líder | Avançado | #02, #05 |

## Guia de Leitura Recomendado

### Para Iniciantes
1. [Arquitetura Geral](docs/01_arquitetura_geral.md) - Entender os componentes
2. [Ciclo de Consenso BFT](docs/02_ciclo_consenso_bft.md) - Entender o fluxo principal
3. [Fluxo de Transações](docs/03_fluxo_transacoes.md) - Entender processamento de TXs
4. [Ciclo Completo de uma Época](docs/07_ciclo_completo_epoca.md) - Ver tudo funcionando junto

### Para Intermediários
1. [Máquina de Estado do Nó](docs/04_node_state_machine.md) - Entender integração
2. [Ciclo de Votação e QC](docs/05_ciclo_votacao_qc.md) - Detalhes de segurança
3. [Tratamento de Falhas e View Change](docs/08_view_change_falhas.md) - Entender resiliência

### Para Avançados
1. [Execução Distribuída](docs/06_execucao_distribuida.md) - Entender complexidade
2. Todos os fluxogramas em detalhe
3. Código-fonte do repositório

## Relações Entre Fluxogramas

```
Arquitetura Geral (#01)
    ↓
Máquina de Estado do Nó (#04)
    ↓
    ├─→ Ciclo de Consenso BFT (#02)
    │       ↓
    │       ├─→ Ciclo de Votação e QC (#05)
    │       └─→ Tratamento de Falhas (#08)
    │
    ├─→ Fluxo de Transações (#03)
    │       ↓
    │       └─→ Execução Distribuída (#06)
    │
    └─→ Ciclo Completo de uma Época (#07)
            (Integra tudo)
```

## Conceitos Fundamentais

### Época (Epoch)
Período de consenso com conjunto fixo de validadores. Cada época tem múltiplos rounds.

### Round
Tentativa de consenso dentro de uma época. Cada round tem um líder eleito deterministicamente.

### Block
Estrutura contendo transações, merkle root, state root, e referência ao bloco anterior.

### Quorum Certificate (QC)
Prova criptográfica de que 2f+1 validadores votaram em um bloco. Contém assinatura agregada BLS12-381.

### Two-Chain Rule
Bloco é commitado quando seu avó tem um QC válido. Garante segurança (safety).

### Vote Locking
Validador não pode votar em blocos conflitantes. Garante segurança.

### Unlock Rule
Permite votar em novo bloco se QC é maior que QC anterior. Garante progresso (liveness).

### State Root
Hash do estado após execução de todas as transações. Determinístico.

### Merkle Root
Hash de todas as transações no bloco. Permite provas de inclusão eficientes.

## Referências

- **Repositório:** https://github.com/flightofthefox/hyperscale-rs
- **Protocolo Base:** HotStuff-2 (Variação de HotStuff)
- **Criptografia:** BLS12-381 (assinaturas agregáveis), Ed25519 (assinaturas rápidas), Blake3 (hashing)
- **Padrão de Consenso:** Byzantine Fault Tolerant (BFT)

---

**Última atualização:** 03 de Fevereiro de 2026
