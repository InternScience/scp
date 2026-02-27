---
name: rare_disease_genetics
description: "Rare Disease Genetic Analysis - Analyze rare disease genetics: Monarch phenotype-disease mapping, ClinVar variants, NCBI gene data, and OpenTargets. Use this skill for rare disease genetics tasks involving get HPO ID by phenotype get joint associated diseases by HPO ID list clinvar search get associated targets by disease efoId. Combines 4 tools from 3 SCP server(s)."
---

# Rare Disease Genetic Analysis

**Discipline**: Rare Disease Genetics | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze rare disease genetics: Monarch phenotype-disease mapping, ClinVar variants, NCBI gene data, and OpenTargets.

## Tools Used

- **`get_HPO_ID_by_phenotype`** from `monarch-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch`
- **`get_joint_associated_diseases_by_HPO_ID_list`** from `monarch-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch`
- **`clinvar_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`
- **`get_associated_targets_by_disease_efoId`** from `opentargets-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets`

## Workflow

1. Get HPO ID for phenotype
2. Find associated diseases
3. Search ClinVar for pathogenic variants
4. Get OpenTargets target associations

## Test Case

### Input
```json
{
    "phenotype": "seizures",
    "hpo_ids": [
        "HP:0001250"
    ]
}
```

### Expected Steps
1. Get HPO ID for phenotype
2. Find associated diseases
3. Search ClinVar for pathogenic variants
4. Get OpenTargets target associations

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "monarch-server": "https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch",
    "search-server": "https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search",
    "opentargets-server": "https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets"
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
    sessions["monarch-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch", "streamable-http")
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")
    sessions["opentargets-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets", "streamable-http")

    # Execute workflow steps
    # Step 1: Get HPO ID for phenotype
    result_1 = await sessions["monarch-server"].call_tool("get_HPO_ID_by_phenotype", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Find associated diseases
    result_2 = await sessions["monarch-server"].call_tool("get_joint_associated_diseases_by_HPO_ID_list", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Search ClinVar for pathogenic variants
    result_3 = await sessions["search-server"].call_tool("clinvar_search", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get OpenTargets target associations
    result_4 = await sessions["opentargets-server"].call_tool("get_associated_targets_by_disease_efoId", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
