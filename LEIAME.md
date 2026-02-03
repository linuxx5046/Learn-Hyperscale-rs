# Aprendendo Rust e Hyperscale com hyperscale-rs

Este repositório contém uma coleção abrangente de guias e fluxogramas projetados para ajudá-lo a aprender Rust e entender sistemas distribuídos usando o projeto **hyperscale-rs** como um exemplo prático.

## 🚀 Início Rápido

Escolha seu caminho de aprendizado:

- **🟢 Iniciante:** Comece com [Fluxograma de Arquitetura Geral](pt/fluxogramas/docs/01_arquitetura_geral.md)
- **🟡 Intermediário:** Comece com [Guia de Aprendizado](pt/guias/GUIA_APRENDIZADO.md)
- **🔴 Avançado:** Comece com [Análise Técnica](pt/guias/hyperscale-analysis.md)

## 📚 Estrutura do Repositório

```
Learn-Hyperscale-rs/
├── README.md                    # Versão em inglês
├── LEIAME.md                    # Este arquivo (Português)
├── LICENSE                      # Licença MIT
├── .gitignore                   # Regras do Git
├── CONTRIBUTING.md              # Guia de contribuição
│
├── en/                          # Conteúdo em inglês
│   ├── INDEX.md                 # Visão geral do conteúdo
│   ├── guides/
│   │   ├── INDEX.md             # Índice de guias
│   │   ├── DETAILED_LEARNING_GUIDE_EN.md
│   │   ├── HYPERSCALE_TECHNICAL_ANALYSIS_EN.md
│   │   └── docs/                # Documentação adicional
│   │
│   └── flowcharts/
│       ├── INDEX.md             # Índice de fluxogramas
│       ├── SUMMARY.md           # Sumário de fluxogramas
│       ├── mermaid/             # Código-fonte dos diagramas (.mmd)
│       ├── images/              # Fluxogramas renderizados (.png)
│       └── docs/                # Documentação de cada fluxograma
│
└── pt/                          # Conteúdo em português
    ├── INDEX.md                 # Visão geral do conteúdo
    ├── guias/
    │   ├── INDEX.md             # Índice de guias
    │   ├── GUIA_APRENDIZADO.md
    │   ├── hyperscale-analysis.md
    │   └── docs/                # Documentação adicional
    │
    └── fluxogramas/
        ├── INDEX.md             # Índice de fluxogramas
        ├── SUMMARY.md           # Sumário de fluxogramas
        ├── mermaid/             # Código-fonte dos diagramas (.mmd)
        ├── images/              # Fluxogramas renderizados (.png)
        └── docs/                # Documentação de cada fluxograma
```

## 📖 Visão Geral do Conteúdo

### Guias

Os guias fornecem um caminho de aprendizado estruturado e progressivo para entender o projeto hyperscale-rs.

- **Guia de Aprendizado** - Guia progressivo do básico até implementação
- **Análise Técnica** - Análise profunda dos componentes técnicos

### Fluxogramas

Os fluxogramas fornecem representações visuais dos principais conceitos. Estão organizados em três formatos complementares:

- **Código-fonte Mermaid** (`.mmd`) - Código-fonte dos diagramas editável
- **Imagens PNG** (`.png`) - Fluxogramas renderizados para visualização rápida
- **Documentação** (`.md`) - Explicação detalhada de cada fluxograma

**8 Fluxogramas Completos:**
1. Arquitetura Geral
2. Ciclo de Consenso BFT
3. Fluxo de Transações
4. Máquina de Estado do Nó
5. Ciclo de Votação e QC
6. Execução Distribuída
7. Ciclo Completo de uma Época
8. Tratamento de Falhas e View Change

## 🎯 Como Usar Este Repositório

### Passo 1: Escolha Seu Idioma
- **Português:** Comece com [pt/INDEX.md](pt/INDEX.md)
- **Inglês:** Comece com [en/INDEX.md](en/INDEX.md)

### Passo 2: Escolha Seu Caminho de Aprendizado
- **Iniciante:** Fluxogramas primeiro, depois guias
- **Intermediário:** Guias primeiro, depois fluxogramas
- **Avançado:** Análise técnica e todos os fluxogramas

### Passo 3: Siga a Ordem de Leitura Recomendada

Cada seção tem uma ordem de leitura recomendada baseada em pré-requisitos e complexidade.

## 🔗 Navegação

### Conteúdo em Português
- [Índice Português](pt/INDEX.md)
- [Índice de Guias](pt/guias/INDEX.md)
- [Índice de Fluxogramas](pt/fluxogramas/INDEX.md)

### Conteúdo em Inglês
- [English Index](en/INDEX.md)
- [Guides Index](en/guides/INDEX.md)
- [Flowcharts Index](en/flowcharts/INDEX.md)

## 📊 Estatísticas

- **Total de Fluxogramas:** 8
- **Total de Guias:** 3+
- **Idiomas:** Português, Inglês
- **Formatos:** Markdown, Mermaid, PNG
- **Tempo Total de Leitura:** ~100+ minutos

## 🤝 Contribuindo

Contribuições são bem-vindas! Consulte [CONTRIBUTING.md](CONTRIBUTING.md) para diretrizes.

## 📝 Licença

Este repositório está licenciado sob a Licença MIT. Veja [LICENSE](LICENSE) para detalhes.

## 🔗 Referências

- **Repositório Hyperscale-RS:** https://github.com/flightofthefox/hyperscale-rs
- **Protocolo Base:** HotStuff-2 (Variação de HotStuff)
- **Padrão de Consenso:** Byzantine Fault Tolerant (BFT)

---

**Última atualização:** 03 de Fevereiro de 2026

**Versão do Repositório:** 2.0 (Reestruturado com padrões profissionais de documentação)
