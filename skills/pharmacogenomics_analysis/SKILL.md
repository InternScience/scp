---
name: pharmacogenomics_analysis
description: "Pharmacogenomics Analysis - Pharmacogenomics analysis: FDA pharmacogenomics info, variant effects, ClinVar pathogenicity, and gene expression. Use this skill for pharmacogenomics tasks involving get pharmacogenomics info by drug name get vep hgvs clinvar search get gene expression across cancers. Combines 4 tools from 4 SCP server(s)."
---

# Pharmacogenomics Analysis

**Discipline**: Pharmacogenomics | **Tools Used**: 4 | **Servers**: 4

## Description

Pharmacogenomics analysis: FDA pharmacogenomics info, variant effects, ClinVar pathogenicity, and gene expression.

## Tools Used

- **`get_pharmacogenomics_info_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_vep_hgvs`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`clinvar_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`
- **`get_gene_expression_across_cancers`** from `tcga-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA`

## Workflow

1. Get FDA pharmacogenomics info
2. Predict effect of CYP2C9 variants
3. Search ClinVar for CYP2C9 variants
4. Check CYP2C9 expression across cancers

## Test Case

### Input
```json
{
    "drug_name": "warfarin",
    "gene": "CYP2C9"
}
```

### Expected Steps
1. Get FDA pharmacogenomics info
2. Predict effect of CYP2C9 variants
3. Search ClinVar for CYP2C9 variants
4. Check CYP2C9 expression across cancers

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
    "ensembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl",
    "search-server": "https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search",
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
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")
    sessions["tcga-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA", "streamable-http")

    # Execute workflow steps
    # Step 1: Get FDA pharmacogenomics info
    result_1 = await sessions["fda-drug-server"].call_tool("get_pharmacogenomics_info_by_drug_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Predict effect of CYP2C9 variants
    result_2 = await sessions["ensembl-server"].call_tool("get_vep_hgvs", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Search ClinVar for CYP2C9 variants
    result_3 = await sessions["search-server"].call_tool("clinvar_search", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Check CYP2C9 expression across cancers
    result_4 = await sessions["tcga-server"].call_tool("get_gene_expression_across_cancers", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
