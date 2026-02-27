---
name: drug_interaction_checker
description: "Drug-Drug Interaction Checker - Check interactions between multiple drugs using FDA interaction data, PubChem compound info, and ChEMBL target overlap analysis. Use this skill for clinical pharmacology tasks involving get drug interactions by drug name get compound by name get target by name. Combines 3 tools from 3 SCP server(s)."
---

# Drug-Drug Interaction Checker

**Discipline**: Clinical Pharmacology | **Tools Used**: 3 | **Servers**: 3

## Description

Check interactions between multiple drugs using FDA interaction data, PubChem compound info, and ChEMBL target overlap analysis.

## Tools Used

- **`get_drug_interactions_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_compound_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`get_target_by_name`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`

## Workflow

1. Get interactions for drug A
2. Get interactions for drug B
3. Compare compound targets from ChEMBL

## Test Case

### Input
```json
{
    "drug_a": "warfarin",
    "drug_b": "aspirin"
}
```

### Expected Steps
1. Get interactions for drug A
2. Get interactions for drug B
3. Compare compound targets from ChEMBL

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "fda-drug-server": "https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug",
    "pubchem-server": "https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem",
    "chembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL"
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
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["pubchem-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem", "streamable-http")
    sessions["chembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL", "streamable-http")

    # Execute workflow steps
    # Step 1: Get interactions for drug A
    result_1 = await sessions["fda-drug-server"].call_tool("get_drug_interactions_by_drug_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get interactions for drug B
    result_2 = await sessions["pubchem-server"].call_tool("get_compound_by_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Compare compound targets from ChEMBL
    result_3 = await sessions["chembl-server"].call_tool("get_target_by_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
