<div align="center">
  <h1>SCP: Scientific Context Protocol</h1>
</div>

> **An open protocol layer for connecting and orchestrating heterogeneous scientific resources, building a global collaborative web of autonomous scientific Agents**

<div align="center">

[![Official Site](https://img.shields.io/badge/Official%20Site-333399.svg?logo=homepage)](https://discovery.intern-ai.org.cn/org/ailab/workspace/iframe?url=https://scphub.intern-ai.org.cn/)&#160;
<a href="https://arxiv.org/abs/2512.24189" target="_blank"><img src="https://img.shields.io/badge/arXiv-b5212f.svg?logo=arxiv" height="21px"></a>
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
  - [Installation and Deployment](#installation-and-deployment)
- [🛠️ Tool Ecosystem](#️-tool-ecosystem)
- [📊 Use Cases](#-use-cases)
- [🔬 SCP vs MCP](#-scp-vs-mcp)
- [🌐 Related Links](#-related-links)
- [📄 License](#-license)
- [Acknowledgments](#acknowledgments)

---

## Overview

SCP (Scientific Context Protocol) is an **open-source standard protocol** designed to accelerate scientific discovery by building a **global web of autonomous scientific agents** that connects heterogeneous resources such as software tools, models, datasets, workflow engines, and experimental instruments.

<p align="center">
  <img src="images/overview.png" alt="SCP overview" width="850">
</p>

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

<p align="center">
  <img src="images/architecture.png" alt="SCP Architecture Overview" width="850">
</p>

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

# install 
pip install mcp

# How to use

There are two ways to use SCP in your workflow:

## Option 1: Build & Manage Your Own
You can set up your own SCP Server and Hub using the provided code. This gives you full control over your deployment and management.

## Option 2: Use Intern-Discovery Platform (Recommended)
Visit the **[SCP Square](https://discovery.intern-ai.org.cn/org/ailab/workspace/iframe?url=https://scphub.intern-ai.org.cn/)** on our **[Intern-Discovery platform](https://discovery.intern-ai.org.cn/org/ailab/workspace/chat)**. We host a ready-to-use SCP Hub with a public registration interface. You can submit your own SCP Server to the square, and it will be integrated into the managed Hub, making it discoverable and accessible across the Intern-Discovery ecosystem.

Choose the approach that best fits your needs—whether you prefer full independence or a collaborative, managed environment.
```


## 🛠️ Tool Ecosystem

SCP has integrated **1,600+ tools**, covering:

- **Biology and Related Technologies** (45.9%)
- **Physics** (21.1%)
- **Chemistry** (11.6%)
- **Mechanics and Materials Science** (8.7%)
- **Mathematics** (8.0%)
- **Information Science and Computing Technology** (4.6%)

🔗 **[View Complete Tool List](https://yankai96.github.io/SCP_Tool_List/)**

## 📊 Use Cases

| Use Case | Description |
|----------|-------------|
| Automated Experimental Protocol Design | Generate executable experimental protocols from natural language objectives |
| Automated Reproduction from PDF Protocols | Extract experimental steps from PDFs and execute them automatically |
| AI-Driven Molecular Screening | Integrate QED scoring, ADMET prediction, and molecular docking |
| Dry-Wet Integrated Protein Engineering | Closed-loop workflow from sequence design to experimental validation |

For detailed case descriptions, refer to the [Technical Report](https://arxiv.org/abs/2512.24189) or [User Cases](https://github.com/InternScience/scp/tree/main/UserCases).

## 🔬 SCP vs MCP

| Feature | MCP | SCP |
|---------|-----|-----|
| Protocol Standardization | General tool invocation | Structured full scientific experiment workflow |
| High-Throughput Experiment Support | No built-in experiment management | Supports batch experiments with context management |
| Multi-Agent Collaboration | Point-to-point communication | Centralized orchestration and task distribution |
| Wet-Lab Equipment Integration | Requires custom adapters | Standardized device drivers and interfaces |

## 🌐 Related Links

- **SCP Tool Plaza**: [Explore 1,600+ integrated tools](https://yankai96.github.io/SCP_Tool_List/)
- **Chinese SCP Documentation**: [Detailed usage guide](https://github.com/InternScience/scp/blob/main/SCP%E4%B8%AD%E6%96%87%E6%89%8B%E5%86%8C.md)
- **Paper & Technical Report**: [SCP design and experimental details]()
- **Community Discussions**: [Questions and discussions](https://www.shlab.org.cn/news/5444176)

## 📄 License
This project is open source under the Apache License 2.0.

## Acknowledgments
SCP is developed by Shanghai Artificial Intelligence Laboratory with support from the open-source community.

If you use SCP, please cite our technical report:
```bibtex
@article{jiang2025scp,
  title={SCP: Accelerating Discovery with a Global Web of Autonomous Scientific Agents},
  author={Jiang, Yankai and Lou, Wenjie and Wang, Lilong and Tang, Zhenyu and Feng, Shiyang and Lu, Jiaxuan and Sun, Haoran and Pan, Yaning and Gu, Shuang and Su, Haoyang and others},
  journal={arXiv preprint arXiv:2512.24189},
  year={2025}
}
```
