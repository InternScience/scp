---
name: snp_functional_analysis
description: "SNP Functional Impact Analysis - Analyze SNP function: VEP prediction, variation details, phenotype association, and literature evidence. Use this skill for functional genomics tasks involving get vep id get variation get phenotype accession pubmed search. Combines 4 tools from 2 SCP server(s)."
---

# SNP Functional Impact Analysis

**Discipline**: Functional Genomics | **Tools Used**: 4 | **Servers**: 2

## Description

Analyze SNP function: VEP prediction, variation details, phenotype association, and literature evidence.

## Tools Used

- **`get_vep_id`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_variation`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`get_phenotype_accession`** from `ensembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl`
- **`pubmed_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`

## Workflow

1. Predict functional effects with VEP
2. Get variant details
3. Get phenotype associations
4. Search PubMed for evidence

## Test Case

### Input
```json
{
    "variant_id": "rs1800497",
    "species": "homo_sapiens"
}
```

### Expected Steps
1. Predict functional effects with VEP
2. Get variant details
3. Get phenotype associations
4. Search PubMed for evidence

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
    sessions["ensembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/12/Origene-Ensembl", "streamable-http")
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")

    # Execute workflow steps
    # Step 1: Predict functional effects with VEP
    result_1 = await sessions["ensembl-server"].call_tool("get_vep_id", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get variant details
    result_2 = await sessions["ensembl-server"].call_tool("get_variation", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get phenotype associations
    result_3 = await sessions["ensembl-server"].call_tool("get_phenotype_accession", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search PubMed for evidence
    result_4 = await sessions["search-server"].call_tool("pubmed_search", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
