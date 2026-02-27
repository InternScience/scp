---
name: substructure_activity_search
description: "Substructure-Activity Relationship - Analyze substructure-activity: ChEMBL substructure search, activity data, PubChem compounds, and similarity. Use this skill for medicinal chemistry tasks involving get substructure by smiles search activity search pubchem by smiles calculate smiles similarity. Combines 4 tools from 3 SCP server(s)."
---

# Substructure-Activity Relationship

**Discipline**: Medicinal Chemistry | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze substructure-activity: ChEMBL substructure search, activity data, PubChem compounds, and similarity.

## Tools Used

- **`get_substructure_by_smiles`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`search_activity`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`search_pubchem_by_smiles`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`calculate_smiles_similarity`** from `server-2` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool`

## Workflow

1. Search ChEMBL by substructure
2. Get bioactivity data for hits
3. Search PubChem for related compounds
4. Compute similarity matrix

## Test Case

### Input
```json
{
    "smiles": "c1ccc2[nH]ccc2c1"
}
```

### Expected Steps
1. Search ChEMBL by substructure
2. Get bioactivity data for hits
3. Search PubChem for related compounds
4. Compute similarity matrix

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "chembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL",
    "pubchem-server": "https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem",
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
    sessions["chembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL", "streamable-http")
    sessions["pubchem-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem", "streamable-http")
    sessions["server-2"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/2/DrugSDA-Tool", "streamable-http")

    # Execute workflow steps
    # Step 1: Search ChEMBL by substructure
    result_1 = await sessions["chembl-server"].call_tool("get_substructure_by_smiles", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get bioactivity data for hits
    result_2 = await sessions["chembl-server"].call_tool("search_activity", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Search PubChem for related compounds
    result_3 = await sessions["pubchem-server"].call_tool("search_pubchem_by_smiles", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Compute similarity matrix
    result_4 = await sessions["server-2"].call_tool("calculate_smiles_similarity", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
