---
name: polypharmacology_analysis
description: "Polypharmacology Analysis - Analyze a drug's multi-target pharmacology: get targets from ChEMBL, functional enrichment from STRING, and pathway links from KEGG. Use this skill for pharmacology tasks involving get target by name get functional enrichment kegg link get mechanism by id. Combines 4 tools from 3 SCP server(s)."
---

# Polypharmacology Analysis

**Discipline**: Pharmacology | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze a drug's multi-target pharmacology: get targets from ChEMBL, functional enrichment from STRING, and pathway links from KEGG.

## Tools Used

- **`get_target_by_name`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_functional_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`kegg_link`** from `kegg-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG`
- **`get_mechanism_by_id`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`

## Workflow

1. Get drug targets from ChEMBL
2. Run functional enrichment on targets
3. Link to KEGG pathways
4. Get mechanism details

## Test Case

### Input
```json
{
    "drug_name": "imatinib"
}
```

### Expected Steps
1. Get drug targets from ChEMBL
2. Run functional enrichment on targets
3. Link to KEGG pathways
4. Get mechanism details

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
    "kegg-server": "https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG"
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
    sessions["kegg-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG", "streamable-http")

    # Execute workflow steps
    # Step 1: Get drug targets from ChEMBL
    result_1 = await sessions["chembl-server"].call_tool("get_target_by_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Run functional enrichment on targets
    result_2 = await sessions["string-server"].call_tool("get_functional_enrichment", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Link to KEGG pathways
    result_3 = await sessions["kegg-server"].call_tool("kegg_link", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get mechanism details
    result_4 = await sessions["chembl-server"].call_tool("get_mechanism_by_id", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
