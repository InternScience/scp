---
name: chemical_structure_comparison
description: "Chemical Structure Comparison - Compare chemical structures: get SMILES, analyze structures, compute similarity, and check PubChem records. Use this skill for cheminformatics tasks involving NameToSMILES ChemicalStructureAnalyzer calculate smiles similarity get compound by name. Combines 4 tools from 4 SCP server(s)."
---

# Chemical Structure Comparison

**Discipline**: Cheminformatics | **Tools Used**: 4 | **Servers**: 4

## Description

Compare chemical structures: get SMILES, analyze structures, compute similarity, and check PubChem records.

## Tools Used

- **`NameToSMILES`** from `server-31` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/31/SciToolAgent-Chem`
- **`ChemicalStructureAnalyzer`** from `server-28` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/28/InternAgent`
- **`calculate_smiles_similarity`** from `server-2` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool`
- **`get_compound_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`

## Workflow

1. Convert names to SMILES
2. Analyze both structures
3. Compute similarity
4. Get PubChem compound data

## Test Case

### Input
```json
{
    "compound_a": "aspirin",
    "compound_b": "ibuprofen"
}
```

### Expected Steps
1. Convert names to SMILES
2. Analyze both structures
3. Compute similarity
4. Get PubChem compound data

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-31": "https://scp.intern-ai.org.cn/api/v1/mcp/31/SciToolAgent-Chem",
    "server-28": "https://scp.intern-ai.org.cn/api/v1/mcp/28/InternAgent",
    "server-2": "https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool",
    "pubchem-server": "https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem"
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
    sessions["server-31"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/31/SciToolAgent-Chem", "sse")
    sessions["server-28"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/28/InternAgent", "sse")
    sessions["server-2"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool", "streamable-http")
    sessions["pubchem-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem", "streamable-http")

    # Execute workflow steps
    # Step 1: Convert names to SMILES
    result_1 = await sessions["server-31"].call_tool("NameToSMILES", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Analyze both structures
    result_2 = await sessions["server-28"].call_tool("ChemicalStructureAnalyzer", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Compute similarity
    result_3 = await sessions["server-2"].call_tool("calculate_smiles_similarity", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get PubChem compound data
    result_4 = await sessions["pubchem-server"].call_tool("get_compound_by_name", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
