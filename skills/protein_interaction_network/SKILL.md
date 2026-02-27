---
name: protein_interaction_network
description: "Protein Interaction Network Analysis - Build protein interaction network: map identifiers with STRING, get PPI network, compute enrichment, and link to KEGG pathways. Use this skill for systems biology tasks involving mapping identifiers get string network interaction get ppi enrichment kegg link. Combines 4 tools from 2 SCP server(s)."
---

# Protein Interaction Network Analysis

**Discipline**: Systems Biology | **Tools Used**: 4 | **Servers**: 2

## Description

Build protein interaction network: map identifiers with STRING, get PPI network, compute enrichment, and link to KEGG pathways.

## Tools Used

- **`mapping_identifiers`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`get_string_network_interaction`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`get_ppi_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`kegg_link`** from `kegg-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG`

## Workflow

1. Map gene names to STRING IDs
2. Get interaction network
3. Compute PPI enrichment
4. Link to KEGG pathways

## Test Case

### Input
```json
{
    "genes": [
        "TP53",
        "BRCA1",
        "MDM2"
    ],
    "species": 9606
}
```

### Expected Steps
1. Map gene names to STRING IDs
2. Get interaction network
3. Compute PPI enrichment
4. Link to KEGG pathways

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
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
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")
    sessions["kegg-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG", "streamable-http")

    # Execute workflow steps
    # Step 1: Map gene names to STRING IDs
    result_1 = await sessions["string-server"].call_tool("mapping_identifiers", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get interaction network
    result_2 = await sessions["string-server"].call_tool("get_string_network_interaction", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Compute PPI enrichment
    result_3 = await sessions["string-server"].call_tool("get_ppi_enrichment", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Link to KEGG pathways
    result_4 = await sessions["kegg-server"].call_tool("kegg_link", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
