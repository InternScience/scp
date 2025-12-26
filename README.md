<div align="center">
  <h1>SCP: Scientific Intelligence Context Protocol</h1>
</div>

> **An open protocol layer for connecting and orchestrating heterogeneous scientific resources, building a global collaborative web of autonomous scientific Agents**

<div align="center">

[![Official Site](https://img.shields.io/badge/Official%20Site-333399.svg?logo=homepage)](https://discovery.intern-ai.org.cn/org/ailab/workspace/iframe?url=https://scphub.intern-ai.org.cn/)&#160;
<a href="https://arxiv.org/pdf" target="_blank"><img src="https://img.shields.io/badge/arXiv-b5212f.svg?logo=arxiv" height="21px"></a>
[![Tool List](https://img.shields.io/badge/tools-1600%2B-brightgreen)](https://yankai96.github.io/SCP_Tool_List/)&#160;
[![GitHub](https://img.shields.io/badge/GitHub-000000?logo=github&logoColor=white)](https://github.com/InternScience/scp)&#160;
</div>

---

## 📖 Table of Contents

- [📖 Table of Contents](#-table-of-contents)
- [Overview](#overview)
- [✨ Key Features](#-key-features)
- [🏗️ Architecture Overview](#️-architecture-overview)
- [🚀 Getting Started](#-getting-started)
  - [1. Installation and Deployment](#1-installation-and-deployment)
  - [2. Register Tools](#2-register-tools)
  - [3. Submit Experiment Requests](#3-submit-experiment-requests)
- [🛠️ Tool Ecosystem](#️-tool-ecosystem)
- [📊 Use Cases](#-use-cases)
- [🔬 SCP vs MCP](#-scp-vs-mcp)
- [🌐 Related Links](#-related-links)
- [📄 License](#-license)
- [🙏 Acknowledgments](#-acknowledgments)

---

## Overview

SCP (Scientific Intelligence Context Protocol) is an **open-source standard protocol** designed to accelerate scientific discovery by building a **global web of autonomous scientific agents** that connects heterogeneous resources such as software tools, models, datasets, workflow engines, and experimental instruments.

SCP provides:

- **Protocol-level connectivity**: Unified description and invocation of **1,600+** scientific resources (tools, models, instruments, etc.)
- **Discovery-as-a-service**: Manages the entire experiment lifecycle through a centralized SCP Hub and distributed SCP Servers

> Developed by **Shanghai Artificial Intelligence Laboratory**, SCP aims to promote cross-institution, cross-platform scientific intelligence collaboration.

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **Unified Protocol Layer** | JSON-based standardized interface for unified invocation of tools, models, and instruments |
| **Intelligent Orchestration** | SCP Hub supports automated planning, execution, and monitoring of multi-step workflows |
| **Dry-Wet Experiment Integration** | Seamless integration of computational tools and experimental devices |
| **Multi-Agent Collaboration** | Supports multiple AI agents working collaboratively in a unified context |
| **Full Experiment Lifecycle Management** | End-to-end traceability from registration, planning, execution to archiving |
| **Security and Access Control** | Fine-grained authentication and authorization mechanisms based on experiments |

---

## 🏗️ Architecture Overview

SCP adopts a **Hub-Spoke architecture**:

    SCP Client (User/Application)
    ↓
    SCP Hub (Central Orchestrator)
    ↓
    SCP Server (Edge Node) → Tools/Models/Instruments


- **SCP Hub**: Central orchestrator responsible for intent parsing, workflow generation, task scheduling, and permission management
- **SCP Server**: Edge nodes that interface with local resources (instruments, databases, models, etc.)
- **SCP Client**: User-facing interface for human researchers or AI scientists

---

## 🚀 Getting Started

### Installation and Deployment

```bash
# Clone the repository
git clone https://github.com/InternScience/scp.git
cd scp

# Start SCP Hub
docker-compose up hub

# Start an example SCP Server
docker-compose up server-example
```

## 🛠️ Tool Ecosystem

SCP has integrated **1,600+ tools**, covering:

- **Biology and Related Technologies** (45.9%)
- **Physics** (21.1%)
- **Chemistry** (11.6%)
- **Mechanics and Materials Science** (8.7%)
- **Mathematics** (8.0%)
- **Information Science and Computing Technology** (4.6%)

🔗 **[View Complete Tool List]()**

## 📊 Use Cases

| Use Case | Description |
|----------|-------------|
| Automated Experimental Protocol Design | Generate executable experimental protocols from natural language objectives |
| Automated Reproduction from PDF Protocols | Extract experimental steps from PDFs and execute them automatically |
| AI-Driven Molecular Screening | Integrate QED scoring, ADMET prediction, and molecular docking |
| Dry-Wet Integrated Protein Engineering | Closed-loop workflow from sequence design to experimental validation |

For detailed case descriptions, refer to the [Technical Report]() or [SCP Documentation]().

## 🔬 SCP vs MCP

| Feature | MCP | SCP |
|---------|-----|-----|
| Protocol Standardization | General tool invocation | Structured full scientific experiment workflow |
| High-Throughput Experiment Support | No built-in experiment management | Supports batch experiments with context management |
| Multi-Agent Collaboration | Point-to-point communication | Centralized orchestration and task distribution |
| Wet-Lab Equipment Integration | Requires custom adapters | Standardized device drivers and interfaces |

## 🌐 Related Links

- **SCP Tool Plaza**: [Explore 1,600+ integrated tools]()
- **SCP Documentation**: [Detailed usage guide and API documentation]()
- **Paper & Technical Report**: [SCP design and experimental details]()
- **Community Discussions**: [Questions and discussions]()
- **Demo Video**: [SCP workflow demonstration]()

## 📄 License
This project is open source under the Apache License 2.0.

## 🙏 Acknowledgments
SCP is developed by Shanghai Artificial Intelligence Laboratory with support from the open-source community.

If you use SCP, please cite our technical report:
```bibtex
@article{jiang2025scp,
  title={SCP: Accelerating Discovery with a Global Web of Autonomous Scientific Agents},
  author={Jiang, Yankai and Lu, Jiaxuan and Liu, Feng and Tan, Cheng and Lou, Wenjie and Sun, Haoran and Wei, Wangxu and Zhang, Bo and Wang, Lilong and Pan, Yaning and Tan, Pan and Wang, Xiaosong and Tang, Zhenyu and Gu, Shuang and Zhou, Dongzhan and Zhou, Bowen and Feng, Shiyang and Su, Haoyang and Ling, Fenghua and Bai, Lei},
  journal={arXiv preprint},
  year={2025}
}
```
