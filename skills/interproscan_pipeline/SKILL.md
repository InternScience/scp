---
name: interproscan_pipeline
description: "InterProScan Analysis Pipeline - Run InterProScan for domain analysis, then enrich with UniProt data and STRING interactions. Use this skill for functional proteomics tasks involving interproscan analyze get uniprotkb entry by accession get functional enrichment query interpro. Combines 4 tools from 4 SCP server(s)."
---

# InterProScan Analysis Pipeline

**Discipline**: Functional Proteomics | **Tools Used**: 4 | **Servers**: 4

## Description

Run InterProScan for domain analysis, then enrich with UniProt data and STRING interactions.

## Tools Used

- **`interproscan_analyze`** from `server-17` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/17/BioInfo-Tools`
- **`get_uniprotkb_entry_by_accession`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_functional_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`query_interpro`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`

## Workflow

1. Run InterProScan
2. Get UniProt entry
3. Run functional enrichment
4. Get InterPro annotations

## Test Case

### Input
```json
{
    "sequence": "MKTIIALSYIFCLVFA",
    "accession": "P04637"
}
```

### Expected Steps
1. Run InterProScan
2. Get UniProt entry
3. Run functional enrichment
4. Get InterPro annotations

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-17": "https://scp.intern-ai.org.cn/api/v1/mcp/17/BioInfo-Tools",
    "uniprot-server": "https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt",
    "string-server": "https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING",
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
    sessions["server-17"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/17/BioInfo-Tools", "streamable-http")
    sessions["uniprot-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt", "streamable-http")
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")

    # Execute workflow steps
    # Step 1: Run InterProScan
    result_1 = await sessions["server-17"].call_tool("interproscan_analyze", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get UniProt entry
    result_2 = await sessions["uniprot-server"].call_tool("get_uniprotkb_entry_by_accession", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Run functional enrichment
    result_3 = await sessions["string-server"].call_tool("get_functional_enrichment", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get InterPro annotations
    result_4 = await sessions["server-1"].call_tool("query_interpro", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
