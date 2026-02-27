---
name: uniprot_deep_analysis
description: "UniProt Deep Protein Analysis - Deep UniProt analysis: entry data, UniRef clusters, UniParc cross-references, and gene-centric view. Use this skill for protein science tasks involving get uniprotkb entry by accession get uniref cluster by id get uniparc entry by upi get gene centric by accession. Combines 4 tools from 1 SCP server(s)."
---

# UniProt Deep Protein Analysis

**Discipline**: Protein Science | **Tools Used**: 4 | **Servers**: 1

## Description

Deep UniProt analysis: entry data, UniRef clusters, UniParc cross-references, and gene-centric view.

## Tools Used

- **`get_uniprotkb_entry_by_accession`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_uniref_cluster_by_id`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_uniparc_entry_by_upi`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_gene_centric_by_accession`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`

## Workflow

1. Get UniProtKB entry
2. Get UniRef cluster
3. Get UniParc entry
4. Get gene-centric data

## Test Case

### Input
```json
{
    "accession": "P04637",
    "uniref_id": "UniRef90_P04637",
    "uniparc_id": "UPI0000000001"
}
```

### Expected Steps
1. Get UniProtKB entry
2. Get UniRef cluster
3. Get UniParc entry
4. Get gene-centric data

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "uniprot-server": "https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt"
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

    # Execute workflow steps
    # Step 1: Get UniProtKB entry
    result_1 = await sessions["uniprot-server"].call_tool("get_uniprotkb_entry_by_accession", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get UniRef cluster
    result_2 = await sessions["uniprot-server"].call_tool("get_uniref_cluster_by_id", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get UniParc entry
    result_3 = await sessions["uniprot-server"].call_tool("get_uniparc_entry_by_upi", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get gene-centric data
    result_4 = await sessions["uniprot-server"].call_tool("get_gene_centric_by_accession", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
