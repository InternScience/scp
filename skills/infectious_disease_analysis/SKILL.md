---
name: infectious_disease_analysis
description: "Infectious Disease Analysis - Analyze infectious disease: virus data, taxonomy, antimicrobial drugs, and resistance literature. Use this skill for infectious disease tasks involving get virus dataset report get taxonomy get mechanism of action by drug name pubmed search. Combines 4 tools from 3 SCP server(s)."
---

# Infectious Disease Analysis

**Discipline**: Infectious Disease | **Tools Used**: 4 | **Servers**: 3

## Description

Analyze infectious disease: virus data, taxonomy, antimicrobial drugs, and resistance literature.

## Tools Used

- **`get_virus_dataset_report`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_taxonomy`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_mechanism_of_action_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`pubmed_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`

## Workflow

1. Get virus genome data
2. Get taxonomy
3. Get drug mechanism
4. Search resistance literature

## Test Case

### Input
```json
{
    "virus_accession": "NC_045512.2",
    "drug": "remdesivir",
    "query": "SARS-CoV-2 resistance"
}
```

### Expected Steps
1. Get virus genome data
2. Get taxonomy
3. Get drug mechanism
4. Search resistance literature

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
    sessions["ncbi-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI", "streamable-http")
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")

    # Execute workflow steps
    # Step 1: Get virus genome data
    result_1 = await sessions["ncbi-server"].call_tool("get_virus_dataset_report", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get taxonomy
    result_2 = await sessions["ncbi-server"].call_tool("get_taxonomy", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get drug mechanism
    result_3 = await sessions["fda-drug-server"].call_tool("get_mechanism_of_action_by_drug_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search resistance literature
    result_4 = await sessions["search-server"].call_tool("pubmed_search", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
