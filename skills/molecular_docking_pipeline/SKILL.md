---
name: molecular_docking_pipeline
description: "Molecular Docking Pipeline - Complete docking workflow: retrieve protein structure, predict binding pockets, prepare receptor, and dock ligand. Use this skill for structural biology tasks involving retrieve protein data by pdbcode run fpocket convert pdb to pdbqt dock quick molecule docking. Combines 4 tools from 2 SCP server(s)."
---

# Molecular Docking Pipeline

**Discipline**: Structural Biology | **Tools Used**: 4 | **Servers**: 2

## Description

Complete docking workflow: retrieve protein structure, predict binding pockets, prepare receptor, and dock ligand.

## Tools Used

- **`retrieve_protein_data_by_pdbcode`** from `server-2` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool`
- **`run_fpocket`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`
- **`convert_pdb_to_pdbqt_dock`** from `server-2` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool`
- **`quick_molecule_docking`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`

## Workflow

1. Download protein structure
2. Predict binding pockets
3. Prepare receptor for docking
4. Perform docking

## Test Case

### Input
```json
{
    "pdb_code": "1AKE",
    "ligand_smiles": "CC(=O)Oc1ccccc1C(=O)O"
}
```

### Expected Steps
1. Download protein structure
2. Predict binding pockets
3. Prepare receptor for docking
4. Perform docking

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-2": "https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool",
    "server-3": "https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model"
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
    sessions["server-2"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool", "streamable-http")
    sessions["server-3"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model", "streamable-http")

    # Execute workflow steps
    # Step 1: Download protein structure
    result_1 = await sessions["server-2"].call_tool("retrieve_protein_data_by_pdbcode", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Predict binding pockets
    result_2 = await sessions["server-3"].call_tool("run_fpocket", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Prepare receptor for docking
    result_3 = await sessions["server-2"].call_tool("convert_pdb_to_pdbqt_dock", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Perform docking
    result_4 = await sessions["server-3"].call_tool("quick_molecule_docking", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
