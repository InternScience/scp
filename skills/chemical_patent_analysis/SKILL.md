---
name: chemical_patent_analysis
description: "Chemical Patent & Novelty Analysis - Analyze chemical novelty: PubChem substructure CAS search, ChEMBL similarity search, compound synonyms, and literature. Use this skill for patent chemistry tasks involving get substructure cas get similarity by smiles get compound synonyms by name search literature. Combines 4 tools from 3 SCP server(s)."
---

# Chemical Patent & Novelty Analysis

**Discipline**: Patent Chemistry | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze chemical novelty: PubChem substructure CAS search, ChEMBL similarity search, compound synonyms, and literature.

## Tools Used

- **`get_substructure_cas`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`get_similarity_by_smiles`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_compound_synonyms_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`search_literature`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`

## Workflow

1. Search CAS by substructure
2. Search ChEMBL by similarity
3. Get compound synonyms
4. Search patent literature

## Test Case

### Input
```json
{
    "smiles": "c1ccc(-c2ccccc2)cc1",
    "compound_name": "biphenyl"
}
```

### Expected Steps
1. Search CAS by substructure
2. Search ChEMBL by similarity
3. Get compound synonyms
4. Search patent literature

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
    "server-1": "https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory"
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
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")

    # Execute workflow steps
    # Step 1: Search CAS by substructure
    result_1 = await sessions["pubchem-server"].call_tool("get_substructure_cas", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Search ChEMBL by similarity
    result_2 = await sessions["chembl-server"].call_tool("get_similarity_by_smiles", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get compound synonyms
    result_3 = await sessions["pubchem-server"].call_tool("get_compound_synonyms_by_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search patent literature
    result_4 = await sessions["server-1"].call_tool("search_literature", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
