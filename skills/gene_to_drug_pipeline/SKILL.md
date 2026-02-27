---
name: gene_to_drug_pipeline
description: "Gene-to-Drug Discovery Pipeline - Full gene-to-drug pipeline: gene lookup, protein structure, binding pocket, virtual screening, and drug-likeness. Use this skill for translational medicine tasks involving get gene metadata by gene name pred protein structure esmfold run fpocket boltz binding affinity calculate mol drug chemistry. Combines 5 tools from 3 SCP server(s)."
---

# Gene-to-Drug Discovery Pipeline

**Discipline**: Translational Medicine | **Tools Used**: 5 | **Servers**: 3

## Description

Full gene-to-drug pipeline: gene lookup, protein structure, binding pocket, virtual screening, and drug-likeness.

## Tools Used

- **`get_gene_metadata_by_gene_name`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`pred_protein_structure_esmfold`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`
- **`run_fpocket`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`
- **`boltz_binding_affinity`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`
- **`calculate_mol_drug_chemistry`** from `server-2` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool`

## Workflow

1. Get gene info from NCBI
2. Predict protein structure
3. Identify binding pockets
4. Predict ligand binding
5. Assess drug-likeness

## Test Case

### Input
```json
{
    "gene": "BRAF",
    "sequence": "MAALSGPGPGA"
}
```

### Expected Steps
1. Get gene info from NCBI
2. Predict protein structure
3. Identify binding pockets
4. Predict ligand binding
5. Assess drug-likeness

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "ncbi-server": "https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI",
    "server-3": "https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model",
    "server-2": "https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool"
}

async def connect(url, transport_type):
    transport = streamablehttp_client(url=url, headers={"SCP-HUB-API-KEY": "<YOUR_SCP_HUB_API_KEY>"})
    read, write, _ = await transport.__aenter__()
    ctx = ClientSession(read, write)
    session = await ctx.__aenter__()
    await session.initialize()
    return session, ctx, transport

def parse(result):
    try:
        if hasattr(result, 'content') and result.content:
            c = result.content[0]
            if hasattr(c, 'text'):
                try: return json.loads(c.text)
                except: return c.text
        return str(result)
    except: return str(result)

async def main():
    # Connect to required servers
    sessions = {}
    sessions["ncbi-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI", "streamable-http")
    sessions["server-3"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model", "streamable-http")
    sessions["server-2"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool", "streamable-http")

    # Execute workflow steps
    # Step 1: Get gene info from NCBI
    result_1 = await sessions["ncbi-server"].call_tool("get_gene_metadata_by_gene_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Predict protein structure
    result_2 = await sessions["server-3"].call_tool("pred_protein_structure_esmfold", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Identify binding pockets
    result_3 = await sessions["server-3"].call_tool("run_fpocket", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Predict ligand binding
    result_4 = await sessions["server-3"].call_tool("boltz_binding_affinity", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Step 5: Assess drug-likeness
    result_5 = await sessions["server-2"].call_tool("calculate_mol_drug_chemistry", arguments={})
    data_5 = parse(result_5)
    print(f"Step 5 result: {json.dumps(data_5, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
