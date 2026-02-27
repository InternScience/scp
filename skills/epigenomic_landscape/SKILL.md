---
name: epigenomic_landscape
description: "Epigenomic Landscape Mapping - Map epigenomic landscape: overlapping features, regulatory elements, binding matrices, and phenotype links. Use this skill for epigenomics tasks involving get overlap region get phenotype region get species binding matrix get track data. Combines 4 tools from 2 SCP server(s)."
---

# Epigenomic Landscape Mapping

**Discipline**: Epigenomics | **Tools Used**: 4 | **Servers**: 2

## Description

Map epigenomic landscape: overlapping features, regulatory elements, binding matrices, and phenotype links.

## Tools Used

- **`get_overlap_region`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_phenotype_region`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_species_binding_matrix`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_track_data`** from `ucsc-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/13/Origene-UCSC`

## Workflow

1. Get overlapping regulatory features
2. Get phenotype associations in region
3. Get binding matrix data
4. Get UCSC epigenomic track data

## Test Case

### Input
```json
{
    "region": "17:43044295-43125370",
    "species": "homo_sapiens",
    "genome": "hg38"
}
```

### Expected Steps
1. Get overlapping regulatory features
2. Get phenotype associations in region
3. Get binding matrix data
4. Get UCSC epigenomic track data

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "ensembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl",
    "ucsc-server": "https://scp.intern-ai.org.cn/api/v1/mcp/13/Origene-UCSC"
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
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")
    sessions["ucsc-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/13/Origene-UCSC", "streamable-http")

    # Execute workflow steps
    # Step 1: Get overlapping regulatory features
    result_1 = await sessions["ensembl-server"].call_tool("get_overlap_region", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get phenotype associations in region
    result_2 = await sessions["ensembl-server"].call_tool("get_phenotype_region", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get binding matrix data
    result_3 = await sessions["ensembl-server"].call_tool("get_species_binding_matrix", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get UCSC epigenomic track data
    result_4 = await sessions["ucsc-server"].call_tool("get_track_data", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
