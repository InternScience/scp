<div align="center">
  <h1>SCP + Claude Code Tutorial</h1>
  <p><strong>A complete guide to setting up the Docker environment, configuring LLM providers, and using SCP Scientific Skills</strong></p>
</div>

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Build the Docker Image](#step-1-build-the-docker-image)
- [Step 2: Run the Container](#step-2-run-the-container)
- [Step 3: Configure cc-switch](#step-3-configure-cc-switch)
- [Step 4: Launch Claude Code](#step-4-launch-claude-code)
- [Step 5: Using SCP Scientific Skills](#step-5-using-scp-scientific-skills)
  - [What Are SCP Skills?](#what-are-scp-skills)
  - [Install Skills into Claude Code](#install-skills-into-claude-code)
  - [Browse Available Skills](#browse-available-skills)
  - [Using a Skill — Step by Step](#using-a-skill--step-by-step)
  - [Skill Examples by Domain](#skill-examples-by-domain)
  - [Running Skills Manually (Without Claude Code)](#running-skills-manually-without-claude-code)
  - [Authentication Setup](#authentication-setup)
- [Supported Models](#supported-models)
- [FAQ](#faq)

---

## Prerequisites

Before you begin, make sure your system has the following installed:

- **Docker** (version 20.10 or higher)
- **Docker Compose** (optional but recommended)
- At least **5 GB** of free disk space

---

## Step 1: Build the Docker Image

### 1.1 Clone the Repository

```bash
git clone https://github.com/InternScience/scp.git
cd scp/docker
```

### 1.2 Build the Image

Run the following command in the project root directory:

```bash
docker build -t claude-code:latest -f .devcontainer/Dockerfile .
```

**Build options explained:**
- `-t claude-code:latest` — assigns a tag to the image
- `-f .devcontainer/Dockerfile` — specifies the Dockerfile path
- Estimated build time: **5–10 minutes** (depending on network speed)

### 1.3 Verify the Image

```bash
docker images | grep claude-code
```

You should see output similar to:

```
claude-code    latest    abc123def456    2 minutes ago    1.2GB
```

---

## Step 2: Run the Container

### 2.1 Start the Container

```bash
docker run -it \
  --name claude-code-container \
  -v $(pwd):/workspace \
  -v claude-history:/commandhistory \
  -p 3000:3000 \
  claude-code:latest
```

**Parameters explained:**
| Flag | Description |
|------|-------------|
| `-it` | Interactive terminal mode |
| `--name` | Container name |
| `-v $(pwd):/workspace` | Mount current directory to the container workspace |
| `-v claude-history:/commandhistory` | Persist command history |
| `-p 3000:3000` | Port mapping (for Web service access if needed) |

### 2.2 Using Docker Compose (Recommended)

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  claude-code:
    image: claude-code:latest
    container_name: claude-code-container
    stdin_open: true
    tty: true
    volumes:
      - ./:/workspace
      - claude-history:/commandhistory
      - claude-config:/home/node/.claude
    ports:
      - "3000:3000"
    environment:
      - TZ=Asia/Shanghai

volumes:
  claude-history:
  claude-config:
```

Start the container:

```bash
docker-compose up -d
docker-compose exec claude-code zsh
```

---

## Step 3: Configure cc-switch

**cc-switch** is a multi-model switching tool that supports major LLM providers worldwide. All configuration is done through an interactive CLI.

### 3.1 Verify Installation

After entering the container, verify that cc-switch is installed:

```bash
cc-switch --version
```

### 3.2 View Help

```bash
# View all commands
cc-switch --help

# View provider subcommand help
cc-switch provider --help
```

**Quick reference:**

```bash
cc-switch provider list              # List all providers
cc-switch provider current           # Show current provider
cc-switch provider switch <id>       # Switch provider
cc-switch provider add               # Add new provider (interactive)
cc-switch provider edit <id>         # Edit existing provider
cc-switch provider duplicate <id>    # Duplicate a provider
cc-switch provider delete <id>       # Delete a provider
cc-switch provider speedtest <id>    # Test API latency
```

### 3.3 Add Model Providers (Interactive)

#### Adding an International Model (e.g., Anthropic Claude)

```bash
cc-switch provider add
```

Follow the interactive prompts:

```
? Enter provider name: anthropic-claude

? Enter API key: sk-ant-xxxxxxxxxxxxxxxxxxxxx

? Enter base URL (optional): https://api.anthropic.com

? Enter model name: claude-3-5-sonnet-20241022

? Enable verbose output? (y/N): n

✅ Provider 'anthropic-claude' added successfully!
```

#### Adding a Chinese Model (e.g., Alibaba Qwen)

```bash
cc-switch provider add
```

```
? Enter provider name: aliyun-qwen

? Enter API key: sk-xxxxxxxxxxxxxxxxxxxxx

? Enter base URL (optional): https://dashscope.aliyuncs.com/compatible-mode/v1

? Enter model name: qwen-max

? Enable verbose output? (y/N): n

✅ Provider 'aliyun-qwen' added successfully!
```

### 3.4 Manage Provider Configurations

**List all providers:**

```bash
cc-switch provider list
```

```
┌────┬──────────────────┬─────────────────────────────┬────────────────────────────────────────┐
│ ID │ Name             │ Model                       │ Base URL                               │
├────┼──────────────────┼─────────────────────────────┼────────────────────────────────────────┤
│ 1  │ anthropic-claude │ claude-3-5-sonnet-20241022  │ https://api.anthropic.com              │
│ 2  │ openai-gpt4      │ gpt-4-turbo                 │ https://api.openai.com/v1              │
│ 3  │ aliyun-qwen      │ qwen-max                    │ https://dashscope.aliyuncs.com/...     │
│ 4* │ deepseek-chat    │ deepseek-chat               │ https://api.deepseek.com/v1            │
│ 5  │ zhipu-glm        │ glm-4                       │ https://open.bigmodel.cn/api/paas/v4   │
└────┴──────────────────┴─────────────────────────────┴────────────────────────────────────────┘

* = Currently active
```

**View current provider:**

```bash
cc-switch provider current
```

```
Current provider: deepseek-chat
Model: deepseek-chat
Base URL: https://api.deepseek.com/v1
```

**Switch provider:**

```bash
# By ID
cc-switch provider switch 1
# Or by name
cc-switch provider switch anthropic-claude
```

```
✅ Switched to provider: anthropic-claude
```

**Edit a provider:**

```bash
cc-switch provider edit 1
```

**Test API latency:**

```bash
cc-switch provider speedtest 1
```

```
Testing provider: anthropic-claude
Model: claude-3-5-sonnet-20241022
Base URL: https://api.anthropic.com

🚀 Sending test request...
⏱️  Response time: 245ms
✅ API connection successful!
```

**Delete a provider:**

```bash
cc-switch provider delete 6
```

```
⚠️  Are you sure you want to delete 'anthropic-claude-backup'? (y/N): y
✅ Provider deleted successfully!
```

---

## Step 4: Launch Claude Code

### 4.1 Start Claude Code

Inside the container, run:

```bash
claude
```

Or specify a working directory:

```bash
claude /workspace/your-project
```

### 4.2 Using Claude Code

After launching, you will see the Claude Code interactive interface:

```
╭─────────────────────────────────────────────────────╮
│                                                     │
│  Welcome to Claude Code!                            │
│                                                     │
│  Current Model: deepseek-chat                       │
│  Provider: https://api.deepseek.com/v1              │
│                                                     │
╰─────────────────────────────────────────────────────╯

You:
```

### 4.3 Switch Models and Restart

```bash
# Switch to Claude
cc-switch provider switch anthropic-claude

# Restart Claude Code
claude
```

---

## Step 5: Using SCP Scientific Skills

Now that Claude Code is running, you can leverage **206 pre-built SCP Scientific Skills** that connect to live scientific computing services. These skills turn Claude Code into a powerful scientific research assistant.

### What Are SCP Skills?

Each SCP Skill is a self-contained `SKILL.md` file that teaches the AI agent how to:

1. **Connect** to one or more SCP Servers via the MCP protocol
2. **Call** specific scientific tools with the correct parameters
3. **Parse** and interpret the results

Skills cover drug discovery, protein engineering, genomics, chemistry, physics, and more — all backed by real, live scientific computing endpoints.

### Install Skills into Claude Code

#### Option A: Clone the Full Repository

If you haven't already cloned the SCP repository:

```bash
cd /workspace
git clone https://github.com/InternScience/scp.git
```

The skills are located in `scp/skills/` — **206 skill folders**, each containing a `SKILL.md`.

#### Option B: Copy Specific Skills

If you only need certain skills, copy them to your project:

```bash
# Copy a single skill
cp -r /workspace/scp/skills/drug_target_identification /workspace/your-project/.claude/skills/

# Copy all skills
cp -r /workspace/scp/skills/ /workspace/your-project/.claude/skills/
```

#### Option C: Point Claude Code to the Skills Directory

Simply work within the SCP repository directory:

```bash
claude /workspace/scp
```

Claude Code will automatically discover and use the skills in the `skills/` directory.

### Browse Available Skills

To see all available skills:

```bash
ls /workspace/scp/skills/
```

Or ask Claude Code directly:

```
You: What SCP skills are available? List them by category.
```

The **206 skills** are organized into 8 domains:

| Domain | Skills | What You Can Do |
|--------|:------:|-----------------|
| 💊 Drug Discovery & Pharmacology | 71 | Target identification, ADMET prediction, virtual screening, drug safety profiling |
| 🧬 Genomics & Genetic Analysis | 41 | Variant pathogenicity, cancer genomics, population genetics, virus genomics |
| 🧬 Protein Science & Engineering | 38 | Structure prediction, binding sites, mutation analysis, antibody design |
| 🧪 Chemistry & Molecular Science | 24 | Structure analysis, fingerprints, SAR, natural products |
| ⚙️ Physics & Engineering | 18 | Circuit analysis, thermodynamics, optics, unit conversion |
| 🔬 Lab Automation & Literature | 7 | Protocol generation, PubMed search, literature mining |
| 🌍 Earth & Environmental Science | 5 | Atmospheric science, oceanography, seismology |
| 📊 Other | 2 | Seismic processing, nanoscale conversion |

### Using a Skill — Step by Step

#### Example 1: Drug Target Identification

Simply describe your research goal in natural language:

```
You: I want to identify potential drug targets for lung cancer (EFO_0000311).
     Find the top targets, get their details from ChEMBL, and retrieve
     protein information from UniProt.
```

Claude Code will automatically use the `drug_target_identification` skill to:
1. Query **OpenTargets** for disease-associated targets
2. Retrieve target details from **ChEMBL**
3. Fetch protein data from **UniProt**

#### Example 2: Protein Structure Analysis

```
You: Analyze the protein structure of p53 (PDB: 1TUP). Download the structure,
     extract chain sequences, calculate structural geometry, and assess quality metrics.
```

Claude Code uses the `protein_structure_analysis` skill to call tools from the **DrugSDA-Tool** server.

#### Example 3: Variant Pathogenicity Assessment

```
You: Assess the pathogenicity of the TP53 variant p.Arg175His.
     Use VEP for effect prediction, check ClinVar, and get phenotype associations.
```

This triggers the `variant_pathogenicity` skill, combining **Ensembl VEP**, **ClinVar**, and **Ensembl phenotype** tools.

#### Example 4: Chemical Structure Comparison

```
You: Compare the chemical structures of aspirin and ibuprofen.
     Convert names to SMILES, analyze their structures, compute similarity,
     and retrieve PubChem data.
```

Claude Code uses the `chemical_structure_comparison` skill across **SciToolAgent-Chem**, **InternAgent**, and **PubChem** servers.

### Skill Examples by Domain

<details>
<summary><b>💊 Drug Discovery — Example Prompts</b></summary>

```
# Lead Compound Optimization
You: Validate the SMILES "CC(=O)Oc1ccccc1C(=O)O", compute drug-likeness metrics,
     predict ADMET properties, and search ChEMBL for bioactivity data.

# Virtual Screening
You: Search PubChem for compounds similar to "c1ccc(-c2ccccc2)cc1",
     compute molecular similarity, filter by drug-likeness, and predict binding affinity.

# Drug Safety Profile
You: Build a comprehensive safety profile for metformin including
     adverse reactions, boxed warnings, drug interactions, and contraindications from FDA.

# Molecular Docking
You: Download protein 1AKE from PDB, predict binding pockets with fpocket,
     prepare the receptor, and dock aspirin into it.
```

</details>

<details>
<summary><b>🧬 Genomics — Example Prompts</b></summary>

```
# Cancer Genomics
You: Analyze TP53 expression across all TCGA cancer types
     and run differential expression analysis for lung cancer.

# Rare Disease
You: Find diseases associated with the phenotype "seizures" (HP:0001250),
     search ClinVar for pathogenic variants, and check OpenTargets for drug targets.

# Genome Annotation
You: Get the NCBI genome annotation for GCF_000001405.40,
     look up BRCA1 in Ensembl, and list UCSC tracks for hg38.
```

</details>

<details>
<summary><b>🧬 Protein Science — Example Prompts</b></summary>

```
# Protein Engineering
You: For the sequence "MKTIIALSYIFCLVFAGKRDEFPSTWYV", predict 3D structure with ESMFold,
     identify functional residues, predict beneficial mutations, and calculate properties.

# Antibody Analysis
You: Get the full UniProt entry for P04637, analyze InterPro domains,
     compute protein parameters, and search ChEMBL for biotherapeutics.

# AlphaFold Pipeline
You: Download the AlphaFold structure for P04637, predict binding pockets,
     extract the sequence, and calculate structure statistics.
```

</details>

<details>
<summary><b>⚙️ Physics & Engineering — Example Prompts</b></summary>

```
# Unit Conversion
You: Convert 100 mm to meters, 0.5 m radius to cm, and 500 nm to micrometers.

# Energy Conversion
You: Convert 5 MeV to Joules, express the result in scientific notation,
     and calculate the conversion error.

# Optics Analysis
You: Calculate the incident photon rate for 1W power at 500nm wavelength,
     determine the frequency range, and compute radiation pressure.
```

</details>

### Running Skills Manually (Without Claude Code)

Each skill contains runnable Python code. You can execute them directly:

```bash
cd /workspace/scp/skills/drug_target_identification
```

Read the `SKILL.md` for the full code example, then run:

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    # Connect to the SCP Server
    transport = streamablehttp_client(
        url="https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets",
        headers={"SCP-HUB-API-KEY": "<YOUR_SCP_HUB_API_KEY>"}
    )
    read, write, _ = await transport.__aenter__()
    ctx = ClientSession(read, write)
    session = await ctx.__aenter__()
    await session.initialize()

    # Call the tool
    result = await session.call_tool(
        "get_associated_targets_by_disease_efoId",
        arguments={"efoId": "EFO_0000311"}
    )

    # Parse result
    content = result.content[0]
    data = json.loads(content.text)
    print(json.dumps(data, indent=2))

    # Cleanup
    await ctx.__aexit__(None, None, None)
    await transport.__aexit__(None, None, None)

asyncio.run(main())
```

### Authentication Setup

All SCP Skills connect to endpoints at `https://scp.intern-ai.org.cn/api/v1/mcp/...` which require an API key.

1. **Get your API key** from the [SCP Platform](https://scp.intern-ai.org.cn)
2. **Replace the placeholder** `<YOUR_SCP_HUB_API_KEY>` in any skill's code with your actual key
3. Or **set it as an environment variable**:

```bash
export SCP_HUB_API_KEY="sk-your-actual-key-here"
```

> **Note:** The API key is required for authentication with the SCP Hub gateway. Each skill's `SKILL.md` contains a placeholder — replace it before use.

---

## Supported Models

cc-switch supports any OpenAI-compatible API provider. Common configurations:

| Provider | Model | Base URL |
|----------|-------|----------|
| Anthropic | claude-3-5-sonnet-20241022 | `https://api.anthropic.com` |
| OpenAI | gpt-4-turbo | `https://api.openai.com/v1` |
| DeepSeek | deepseek-chat | `https://api.deepseek.com/v1` |
| Alibaba Qwen | qwen-max | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| Zhipu GLM | glm-4 | `https://open.bigmodel.cn/api/paas/v4` |

---

## FAQ

**Q: How do I know which skill to use?**
A: Simply describe your research goal in natural language to Claude Code — it will automatically select and invoke the appropriate skill(s). You can also browse all 206 skills in the [`skills/`](https://github.com/InternScience/scp/tree/main/skills) directory.

**Q: Can I use skills without Claude Code?**
A: Yes. Each `SKILL.md` contains standalone Python code that you can copy and run directly. You only need the `mcp` Python package (`pip install mcp`).

**Q: What if a tool call times out?**
A: Some scientific computations (e.g., TCGA differential expression, protein structure prediction) may take longer. Increase the timeout in your code or retry.

**Q: Can I create my own SCP Skills?**
A: Absolutely. Create a new folder under `skills/` with a `SKILL.md` file following the same format. Refer to existing skills as templates.

**Q: How do I update skills?**
A: Pull the latest from the repository:
```bash
cd /workspace/scp
git pull origin main
```

---

<div align="center">
  <p><strong>Made with ❤️ by Shanghai AI Laboratory</strong></p>
  <p>
    <a href="https://github.com/InternScience/scp">GitHub</a> ·
    <a href="https://arxiv.org/abs/2512.24189">Paper</a> ·
    <a href="https://scp.intern-ai.org.cn">SCP Platform</a>
  </p>
</div>
