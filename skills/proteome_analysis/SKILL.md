---
name: proteome_analysis
description: "Proteome-Level Analysis - Analyze at proteome level: get proteome from UniProt, gene-centric view, functional annotation from STRING. Use this skill for proteomics tasks involving get proteome by id get gene centric by proteome get functional annotation. Combines 3 tools from 2 SCP server(s)."
---

# Proteome-Level Analysis

**Discipline**: Proteomics | **Tools Used**: 3 | **Servers**: 2

## Description

Analyze at proteome level: get proteome from UniProt, gene-centric view, functional annotation from STRING.

## Tools Used

- **`get_proteome_by_id`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_gene_centric_by_proteome`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_functional_annotation`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`

## Workflow

1. Get human proteome info
2. Get gene-centric view
3. Run functional annotation on key proteins

## Test Case

### Input
```json
{
    "proteome_id": "UP000005640"
}
```

### Expected Steps
1. Get human proteome info
2. Get gene-centric view
3. Run functional annotation on key proteins

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "uniprot-server": "https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt",
    "string-server": "https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING"
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
    sessions["uniprot-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt", "streamable-http")
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")

    # Execute workflow steps
    # Step 1: Get human proteome info
    result_1 = await sessions["uniprot-server"].call_tool("get_proteome_by_id", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get gene-centric view
    result_2 = await sessions["uniprot-server"].call_tool("get_gene_centric_by_proteome", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Run functional annotation on key proteins
    result_3 = await sessions["string-server"].call_tool("get_functional_annotation", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
