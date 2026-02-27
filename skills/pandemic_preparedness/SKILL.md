---
name: pandemic_preparedness
description: "Pandemic Preparedness Analysis - Pandemic analysis: virus genome, taxonomy, drug candidates, and literature intelligence. Use this skill for public health tasks involving get virus dataset report get virus by taxon genome get mechanism of action by drug name tavily search search literature. Combines 5 tools from 4 SCP server(s)."
---

# Pandemic Preparedness Analysis

**Discipline**: Public Health | **Tools Used**: 5 | **Servers**: 4

## Description

Pandemic analysis: virus genome, taxonomy, drug candidates, and literature intelligence.

## Tools Used

- **`get_virus_dataset_report`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_virus_by_taxon_genome`** from `ncbi-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/9/Origene-NCBI`
- **`get_mechanism_of_action_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`tavily_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`
- **`search_literature`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`

## Workflow

1. Get virus genome data
2. Get virus by taxon
3. Get antiviral mechanism
4. Search latest news
5. Search academic literature

## Test Case

### Input
```json
{
    "virus_accession": "NC_045512.2",
    "taxon": "2697049",
    "drug": "paxlovid"
}
```

### Expected Steps
1. Get virus genome data
2. Get virus by taxon
3. Get antiviral mechanism
4. Search latest news
5. Search academic literature

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
    "search-server": "https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search",
    "server-1": "https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory"
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
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")

    # Execute workflow steps
    # Step 1: Get virus genome data
    result_1 = await sessions["ncbi-server"].call_tool("get_virus_dataset_report", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get virus by taxon
    result_2 = await sessions["ncbi-server"].call_tool("get_virus_by_taxon_genome", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get antiviral mechanism
    result_3 = await sessions["fda-drug-server"].call_tool("get_mechanism_of_action_by_drug_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search latest news
    result_4 = await sessions["search-server"].call_tool("tavily_search", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Step 5: Search academic literature
    result_5 = await sessions["server-1"].call_tool("search_literature", arguments={})
    data_5 = parse(result_5)
    print(f"Step 5 result: {json.dumps(data_5, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
