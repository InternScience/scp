---
name: go_term_analysis
description: "Gene Ontology Analysis - Analyze GO terms: ChEMBL GO slim, STRING functional enrichment, STRING annotation, and Ensembl ontology. Use this skill for functional genomics tasks involving get go slim by id get functional enrichment get functional annotation get ontology name. Combines 4 tools from 3 SCP server(s)."
---

# Gene Ontology Analysis

**Discipline**: Functional Genomics | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze GO terms: ChEMBL GO slim, STRING functional enrichment, STRING annotation, and Ensembl ontology.

## Tools Used

- **`get_go_slim_by_id`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_functional_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`get_functional_annotation`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`get_ontology_name`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`

## Workflow

1. Get ChEMBL GO slim
2. Run STRING enrichment
3. Get STRING annotations
4. Get Ensembl ontology details

## Test Case

### Input
```json
{
    "go_id": "GO:0005515",
    "genes": "TP53",
    "species": 9606
}
```

### Expected Steps
1. Get ChEMBL GO slim
2. Run STRING enrichment
3. Get STRING annotations
4. Get Ensembl ontology details

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
    "string-server": "https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING",
    "ensembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl"
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
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")

    # Execute workflow steps
    # Step 1: Get ChEMBL GO slim
    result_1 = await sessions["chembl-server"].call_tool("get_go_slim_by_id", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Run STRING enrichment
    result_2 = await sessions["string-server"].call_tool("get_functional_enrichment", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get STRING annotations
    result_3 = await sessions["string-server"].call_tool("get_functional_annotation", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get Ensembl ontology details
    result_4 = await sessions["ensembl-server"].call_tool("get_ontology_name", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
