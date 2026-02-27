---
name: multiomics_integration
description: "Multi-Omics Integration - Integrate multi-omics: gene expression, protein data, pathway enrichment, and metabolic pathways. Use this skill for multi-omics tasks involving get gene expression across cancers get uniprotkb entry by accession get functional enrichment kegg get. Combines 4 tools from 4 SCP server(s)."
---

# Multi-Omics Integration

**Discipline**: Multi-Omics | **Tools Used**: 4 | **Servers**: 4

## Description

Integrate multi-omics: gene expression, protein data, pathway enrichment, and metabolic pathways.

## Tools Used

- **`get_gene_expression_across_cancers`** from `tcga-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA`
- **`get_uniprotkb_entry_by_accession`** from `uniprot-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt`
- **`get_functional_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`
- **`kegg_get`** from `kegg-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG`

## Workflow

1. Get transcriptomic data
2. Get proteomic data
3. Run pathway enrichment
4. Get metabolic pathway details

## Test Case

### Input
```json
{
    "gene": "TP53",
    "accession": "P04637",
    "pathway": "hsa04115"
}
```

### Expected Steps
1. Get transcriptomic data
2. Get proteomic data
3. Run pathway enrichment
4. Get metabolic pathway details

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "tcga-server": "https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA",
    "uniprot-server": "https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt",
    "string-server": "https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING",
    "kegg-server": "https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG"
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
    sessions["tcga-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/11/Origene-TCGA", "streamable-http")
    sessions["uniprot-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/10/Origene-UniProt", "streamable-http")
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")
    sessions["kegg-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG", "streamable-http")

    # Execute workflow steps
    # Step 1: Get transcriptomic data
    result_1 = await sessions["tcga-server"].call_tool("get_gene_expression_across_cancers", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get proteomic data
    result_2 = await sessions["uniprot-server"].call_tool("get_uniprotkb_entry_by_accession", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Run pathway enrichment
    result_3 = await sessions["string-server"].call_tool("get_functional_enrichment", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get metabolic pathway details
    result_4 = await sessions["kegg-server"].call_tool("kegg_get", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
