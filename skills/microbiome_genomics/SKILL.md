---
name: microbiome_genomics
description: "Microbiome Genomics Analysis - Analyze microbial genome: NCBI genome data, taxonomy, KEGG metabolic pathways, and annotation. Use this skill for metagenomics tasks involving get genome dataset report by taxon get taxonomy kegg find get genome annotation report. Combines 4 tools from 2 SCP server(s)."
---

# Microbiome Genomics Analysis

**Discipline**: Metagenomics | **Tools Used**: 4 | **Servers**: 2

## Description

Analyze microbial genome: NCBI genome data, taxonomy, KEGG metabolic pathways, and annotation.

## Tools Used

- **`get_genome_dataset_report_by_taxon`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_taxonomy`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`kegg_find`** from `kegg-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG`
- **`get_genome_annotation_report`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`

## Workflow

1. Get genome dataset for E. coli
2. Get taxonomic classification
3. Find KEGG metabolic pathways
4. Get genome annotation

## Test Case

### Input
```json
{
    "taxon": "Escherichia coli",
    "accession": "GCF_000005845.2"
}
```

### Expected Steps
1. Get genome dataset for E. coli
2. Get taxonomic classification
3. Find KEGG metabolic pathways
4. Get genome annotation

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
    sessions["ncbi-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI", "streamable-http")
    sessions["kegg-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/5/Origene-KEGG", "streamable-http")

    # Execute workflow steps
    # Step 1: Get genome dataset for E. coli
    result_1 = await sessions["ncbi-server"].call_tool("get_genome_dataset_report_by_taxon", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get taxonomic classification
    result_2 = await sessions["ncbi-server"].call_tool("get_taxonomy", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Find KEGG metabolic pathways
    result_3 = await sessions["kegg-server"].call_tool("kegg_find", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get genome annotation
    result_4 = await sessions["ncbi-server"].call_tool("get_genome_annotation_report", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
