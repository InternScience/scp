---
name: orphan_drug_analysis
description: "Orphan Drug & Rare Disease Analysis - Analyze orphan drugs: Monarch disease phenotypes, OpenTargets targets, FDA drug data, and clinical studies. Use this skill for orphan drug development tasks involving get joint associated diseases by HPO ID list get associated targets by disease efoId get clinical studies info by drug name pubmed search. Combines 4 tools from 4 SCP server(s)."
---

# Orphan Drug & Rare Disease Analysis

**Discipline**: Orphan Drug Development | **Tools Used**: 4 | **Servers**: 4

## Description

Analyze orphan drugs: Monarch disease phenotypes, OpenTargets targets, FDA drug data, and clinical studies.

## Tools Used

- **`get_joint_associated_diseases_by_HPO_ID_list`** from `monarch-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch`
- **`get_associated_targets_by_disease_efoId`** from `opentargets-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets`
- **`get_clinical_studies_info_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`pubmed_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`

## Workflow

1. Map phenotypes to diseases
2. Find drug targets
3. Get clinical studies
4. Search literature

## Test Case

### Input
```json
{
    "hpo_ids": [
        "HP:0001250"
    ],
    "disease_efo": "MONDO_0010075",
    "query": "orphan drug seizure disorder"
}
```

### Expected Steps
1. Map phenotypes to diseases
2. Find drug targets
3. Get clinical studies
4. Search literature

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
    "opentargets-server": "https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets",
    "fda-drug-server": "https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug",
    "search-server": "https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search"
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
    sessions["opentargets-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets", "streamable-http")
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")

    # Execute workflow steps
    # Step 1: Map phenotypes to diseases
    result_1 = await sessions["monarch-server"].call_tool("get_joint_associated_diseases_by_HPO_ID_list", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Find drug targets
    result_2 = await sessions["opentargets-server"].call_tool("get_associated_targets_by_disease_efoId", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get clinical studies
    result_3 = await sessions["fda-drug-server"].call_tool("get_clinical_studies_info_by_drug_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search literature
    result_4 = await sessions["search-server"].call_tool("pubmed_search", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
