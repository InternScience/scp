---
name: population_genetics
description: "Population Genetics Analysis - Analyze population genetics: Ensembl variation populations, linkage disequilibrium, and variant frequency data. Use this skill for population genetics tasks involving get info variation populations get ld get variation get variant recoder. Combines 4 tools from 1 SCP server(s)."
---

# Population Genetics Analysis

**Discipline**: Population Genetics | **Tools Used**: 4 | **Servers**: 1

## Description

Analyze population genetics: Ensembl variation populations, linkage disequilibrium, and variant frequency data.

## Tools Used

- **`get_info_variation_populations`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_ld`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_variation`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_variant_recoder`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`

## Workflow

1. Get variation populations
2. Calculate LD for variant
3. Get variant details
4. Recode variant identifiers

## Test Case

### Input
```json
{
    "variant_id": "rs699",
    "species": "homo_sapiens",
    "population": "1000GENOMES:phase_3:CEU"
}
```

### Expected Steps
1. Get variation populations
2. Calculate LD for variant
3. Get variant details
4. Recode variant identifiers

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
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
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")

    # Execute workflow steps
    # Step 1: Get variation populations
    result_1 = await sessions["ensembl-server"].call_tool("get_info_variation_populations", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Calculate LD for variant
    result_2 = await sessions["ensembl-server"].call_tool("get_ld", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get variant details
    result_3 = await sessions["ensembl-server"].call_tool("get_variation", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Recode variant identifiers
    result_4 = await sessions["ensembl-server"].call_tool("get_variant_recoder", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
