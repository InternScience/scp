---
name: cancer_therapy_design
description: "Cancer Therapy Design - Design cancer therapy: identify targets, find drugs, check safety, and analyze differential expression. Use this skill for oncology tasks involving get associated targets by disease efoId get associated drugs by target name get adverse reactions by drug name tcga differential expression analysis. Combines 4 tools from 3 SCP server(s)."
---

# Cancer Therapy Design

**Discipline**: Oncology | **Tools Used**: 4 | **Servers**: 3

## Description

Design cancer therapy: identify targets, find drugs, check safety, and analyze differential expression.

## Tools Used

- **`get_associated_targets_by_disease_efoId`** from `opentargets-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets`
- **`get_associated_drugs_by_target_name`** from `opentargets-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets`
- **`get_adverse_reactions_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`tcga_differential_expression_analysis`** from `tcga-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA`

## Workflow

1. Find cancer targets
2. Get associated drugs
3. Check drug safety
4. Analyze differential expression

## Test Case

### Input
```json
{
    "disease_efo": "EFO_0000311",
    "query": "lung adenocarcinoma drug targets"
}
```

### Expected Steps
1. Find cancer targets
2. Get associated drugs
3. Check drug safety
4. Analyze differential expression

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "opentargets-server": "https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets",
    "fda-drug-server": "https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug",
    "tcga-server": "https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA"
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
    sessions["opentargets-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets", "streamable-http")
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["tcga-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA", "streamable-http")

    # Execute workflow steps
    # Step 1: Find cancer targets
    result_1 = await sessions["opentargets-server"].call_tool("get_associated_targets_by_disease_efoId", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get associated drugs
    result_2 = await sessions["opentargets-server"].call_tool("get_associated_drugs_by_target_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Check drug safety
    result_3 = await sessions["fda-drug-server"].call_tool("get_adverse_reactions_by_drug_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Analyze differential expression
    result_4 = await sessions["tcga-server"].call_tool("tcga_differential_expression_analysis", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
