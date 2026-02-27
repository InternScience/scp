---
name: gene_disease_association
description: "Gene-Disease Association Analysis - Analyze gene-disease associations: NCBI gene metadata, OpenTargets disease associations, TCGA expression, and Monarch phenotypes. Use this skill for medical genetics tasks involving get gene metadata by gene name get associated targets by disease efoId get gene expression across cancers get joint associated diseases by HPO ID list. Combines 4 tools from 4 SCP server(s)."
---

# Gene-Disease Association Analysis

**Discipline**: Medical Genetics | **Tools Used**: 4 | **Servers**: 4

## Description

Analyze gene-disease associations: NCBI gene metadata, OpenTargets disease associations, TCGA expression, and Monarch phenotypes.

## Tools Used

- **`get_gene_metadata_by_gene_name`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_associated_targets_by_disease_efoId`** from `opentargets-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets`
- **`get_gene_expression_across_cancers`** from `tcga-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA`
- **`get_joint_associated_diseases_by_HPO_ID_list`** from `monarch-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch`

## Workflow

1. Get gene metadata from NCBI
2. Get disease-target associations from OpenTargets
3. Analyze TCGA cancer expression
4. Check Monarch disease associations

## Test Case

### Input
```json
{
    "gene_name": "TP53",
    "disease_efo": "EFO_0000311"
}
```

### Expected Steps
1. Get gene metadata from NCBI
2. Get disease-target associations from OpenTargets
3. Analyze TCGA cancer expression
4. Check Monarch disease associations

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "ncbi-server": "https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI",
    "opentargets-server": "https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets",
    "tcga-server": "https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA",
    "monarch-server": "https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch"
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
    sessions["ncbi-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI", "streamable-http")
    sessions["opentargets-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/15/Origene-OpenTargets", "streamable-http")
    sessions["tcga-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA", "streamable-http")
    sessions["monarch-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/16/Origene-Monarch", "streamable-http")

    # Execute workflow steps
    # Step 1: Get gene metadata from NCBI
    result_1 = await sessions["ncbi-server"].call_tool("get_gene_metadata_by_gene_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get disease-target associations from OpenTargets
    result_2 = await sessions["opentargets-server"].call_tool("get_associated_targets_by_disease_efoId", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Analyze TCGA cancer expression
    result_3 = await sessions["tcga-server"].call_tool("get_gene_expression_across_cancers", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Check Monarch disease associations
    result_4 = await sessions["monarch-server"].call_tool("get_joint_associated_diseases_by_HPO_ID_list", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
