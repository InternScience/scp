---
name: compound_database_crossref
description: "Cross-Database Compound Lookup - Cross-reference compound across databases: PubChem, ChEMBL, KEGG, and CAS number lookup. Use this skill for chemical information tasks involving get compound by name get molecule by name kegg find CASToPrice. Combines 4 tools from 4 SCP server(s)."
---

# Cross-Database Compound Lookup

**Discipline**: Chemical Information | **Tools Used**: 4 | **Servers**: 4

## Description

Cross-reference compound across databases: PubChem, ChEMBL, KEGG, and CAS number lookup.

## Tools Used

- **`get_compound_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`get_molecule_by_name`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`kegg_find`** from `kegg-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG`
- **`CASToPrice`** from `server-30` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/30/SciToolAgent-Mat`

## Workflow

1. Get PubChem entry
2. Get ChEMBL molecule entry
3. Search KEGG
4. Look up CAS number and pricing

## Test Case

### Input
```json
{
    "compound_name": "aspirin"
}
```

### Expected Steps
1. Get PubChem entry
2. Get ChEMBL molecule entry
3. Search KEGG
4. Look up CAS number and pricing

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "pubchem-server": "https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem",
    "chembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL",
    "kegg-server": "https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG",
    "server-30": "https://scp.intern-ai.org.cn/api/v1/mcp/30/SciToolAgent-Mat"
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
    sessions["pubchem-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem", "streamable-http")
    sessions["chembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL", "streamable-http")
    sessions["kegg-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG", "streamable-http")
    sessions["server-30"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/30/SciToolAgent-Mat", "sse")

    # Execute workflow steps
    # Step 1: Get PubChem entry
    result_1 = await sessions["pubchem-server"].call_tool("get_compound_by_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get ChEMBL molecule entry
    result_2 = await sessions["chembl-server"].call_tool("get_molecule_by_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Search KEGG
    result_3 = await sessions["kegg-server"].call_tool("kegg_find", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Look up CAS number and pricing
    result_4 = await sessions["server-30"].call_tool("CASToPrice", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
