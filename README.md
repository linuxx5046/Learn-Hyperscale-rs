# Learning Rust and Hyperscale with hyperscale-rs

This repository contains a comprehensive collection of guides and flowcharts designed to help you learn Rust and understand distributed systems using the **hyperscale-rs** project as a practical example.

## 🚀 Quick Start

Choose your learning path:

- **🟢 Beginner:** Start with [General Architecture Flowchart](en/flowcharts/docs/01_general_architecture.md)
- **🟡 Intermediate:** Start with [Learning Guide](en/guides/DETAILED_LEARNING_GUIDE_EN.md)
- **🔴 Advanced:** Start with [Technical Analysis](en/guides/HYPERSCALE_TECHNICAL_ANALYSIS_EN.md)

## 📚 Repository Structure

```
Learn-Hyperscale-rs/
├── README.md                    # This file (English)
├── LEIAME.md                    # Portuguese version
├── LICENSE                      # MIT License
├── .gitignore                   # Git ignore rules
├── CONTRIBUTING.md              # Contribution guidelines
│
├── en/                          # English content
│   ├── INDEX.md                 # English content overview
│   ├── guides/
│   │   ├── INDEX.md             # Guides index
│   │   ├── DETAILED_LEARNING_GUIDE_EN.md
│   │   ├── HYPERSCALE_TECHNICAL_ANALYSIS_EN.md
│   │   ├── The Complete Learning Guide_ Demystifying Hyperscale-RS.md
│   │   └── docs/                # Additional guide documentation
│   │
│   └── flowcharts/
│       ├── INDEX.md             # Flowcharts index
│       ├── SUMMARY.md           # Flowcharts summary
│       ├── mermaid/             # Source diagrams (.mmd)
│       │   ├── 01_general_architecture.mmd
│       │   ├── 02_bft_consensus_cycle.mmd
│       │   └── ... (8 total)
│       ├── images/              # Rendered flowcharts (.png)
│       │   ├── 01_general_architecture.png
│       │   ├── 02_bft_consensus_cycle.png
│       │   └── ... (8 total)
│       └── docs/                # Flowchart documentation
│           ├── 01_general_architecture.md
│           └── ... (documentation for each flowchart)
│
└── pt/                          # Portuguese content (Português)
    ├── INDEX.md                 # Portuguese content overview
    ├── guias/
    │   ├── INDEX.md             # Guides index
    │   ├── GUIA_APRENDIZADO.md
    │   ├── hyperscale-analysis.md
    │   ├── 🎓 Guia de Aprendizado Definitivo_ Desvendando o Hyperscale-RS.md
    │   └── docs/                # Additional guide documentation
    │
    └── fluxogramas/
        ├── INDEX.md             # Flowcharts index
        ├── SUMMARY.md           # Flowcharts summary
        ├── mermaid/             # Source diagrams (.mmd)
        │   ├── 01_arquitetura_geral.mmd
        │   ├── 02_ciclo_consenso_bft.mmd
        │   └── ... (8 total)
        ├── images/              # Rendered flowcharts (.png)
        │   ├── 01_arquitetura_geral.png
        │   ├── 02_ciclo_consenso_bft.png
        │   └── ... (8 total)
        └── docs/                # Flowchart documentation
            ├── 01_arquitetura_geral.md
            └── ... (documentation for each flowchart)
```

## 📖 Content Overview

### Guides

The guides provide a structured and progressive learning path to understand the hyperscale-rs project. They cover topics from fundamental concepts to advanced implementation details.

- **Learning Guide** - Progressive guide from basics to implementation
- **Technical Analysis** - Deep dive into technical components
- **Complete Guide** - Comprehensive overview

### Flowcharts

The flowcharts provide visual representations of the main concepts and workflows. They are organized in three complementary formats:

- **Mermaid Source** (`.mmd`) - Editable diagram source code
- **PNG Images** (`.png`) - Rendered flowcharts for quick viewing
- **Documentation** (`.md`) - Detailed explanation of each flowchart

**8 Complete Flowcharts:**
1. General Architecture
2. BFT Consensus Cycle
3. Transaction Flow
4. Node State Machine
5. Voting and QC Cycle
6. Distributed Execution
7. Complete Epoch Cycle
8. Failure Handling and View Change

## 🎯 How to Use This Repository

### Step 1: Choose Your Language
- **English:** Start with [en/INDEX.md](en/INDEX.md)
- **Portuguese:** Start with [pt/INDEX.md](pt/INDEX.md)

### Step 2: Choose Your Learning Path
- **Beginner:** Flowcharts first, then guides
- **Intermediate:** Guides first, then flowcharts
- **Advanced:** Technical analysis and all flowcharts

### Step 3: Follow the Recommended Reading Order

Each section has a recommended reading order based on prerequisites and complexity.

### Step 4: Consult the Source Code

After understanding the concepts through guides and flowcharts, explore the actual implementation:
- Repository: https://github.com/flightofthefox/hyperscale-rs

## 🔑 Key Concepts

### Epoch
A consensus period with a fixed set of validators. Each epoch has multiple rounds.

### Round
A consensus attempt within an epoch. Each round has a deterministically elected leader.

### Quorum Certificate (QC)
Cryptographic proof that 2f+1 validators voted on a block. Contains BLS12-381 aggregated signature.

### Two-Chain Rule
A block is committed when its grandparent has a valid QC. Ensures safety.

### Byzantine Fault Tolerance (BFT)
The system can tolerate up to f Byzantine (malicious) validators, where n = 3f + 1 total validators.

## 🔗 Navigation

### English Content
- [English Index](en/INDEX.md)
- [Guides Index](en/guides/INDEX.md)
- [Flowcharts Index](en/flowcharts/INDEX.md)

### Portuguese Content
- [Índice Português](pt/INDEX.md)
- [Índice de Guias](pt/guias/INDEX.md)
- [Índice de Fluxogramas](pt/fluxogramas/INDEX.md)

## 📊 Statistics

- **Total Flowcharts:** 8
- **Total Guides:** 3+
- **Languages:** English, Portuguese
- **Formats:** Markdown, Mermaid, PNG
- **Total Reading Time:** ~100+ minutes

## 🤝 Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📝 License

This repository is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## 🔗 References

- **Hyperscale-RS Repository:** https://github.com/flightofthefox/hyperscale-rs
- **Base Protocol:** HotStuff-2 (Variation of HotStuff)
- **Consensus Pattern:** Byzantine Fault Tolerant (BFT)

## 📧 Questions?

If you have questions or suggestions, please open an issue on GitHub.

---

**Last updated:** February 3, 2026

**Repository Version:** 2.0 (Restructured with professional documentation standards)
