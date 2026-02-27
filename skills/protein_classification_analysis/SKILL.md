---
name: protein_classification_analysis
description: "Protein Classification Analysis - Classify protein: ChEMBL protein classification, UniProt entry, InterPro domains, and Ensembl biotypes. Use this skill for protein science tasks involving search protein classification get uniprotkb entry by accession query interpro get info biotypes. Combines 4 tools from 4 SCP server(s)."
---

# Protein Classification Analysis

**Discipline**: Protein Science | **Tools Used**: 4 | **Servers**: 4

## Description

Classify protein: ChEMBL protein classification, UniProt entry, InterPro domains, and Ensembl biotypes.

## Tools Used

- **`search_protein_classification`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_uniprotkb_entry_by_accession`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`query_interpro`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`get_info_biotypes`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`

## Workflow

1. Search ChEMBL protein classification
2. Get UniProt entry
3. Get InterPro domains
4. Get Ensembl biotypes

## Test Case

### Input
```json
{
    "protein": "kinase",
    "accession": "P04637",
    "species": "homo_sapiens"
}
```

### Expected Steps
1. Search ChEMBL protein classification
2. Get UniProt entry
3. Get InterPro domains
4. Get Ensembl biotypes

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
    "uniprot-server": "https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt",
    "server-1": "https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory",
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
    sessions["uniprot-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt", "streamable-http")
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")

    # Execute workflow steps
    # Step 1: Search ChEMBL protein classification
    result_1 = await sessions["chembl-server"].call_tool("search_protein_classification", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get UniProt entry
    result_2 = await sessions["uniprot-server"].call_tool("get_uniprotkb_entry_by_accession", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get InterPro domains
    result_3 = await sessions["server-1"].call_tool("query_interpro", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get Ensembl biotypes
    result_4 = await sessions["ensembl-server"].call_tool("get_info_biotypes", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
