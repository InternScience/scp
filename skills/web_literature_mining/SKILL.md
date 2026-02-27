---
name: web_literature_mining
description: "Scientific Literature Mining - Mine scientific literature: PubMed search, arXiv search, web search, and Tavily deep search. Use this skill for scientific informatics tasks involving pubmed search search literature search web tavily search. Combines 4 tools from 2 SCP server(s)."
---

# Scientific Literature Mining

**Discipline**: Scientific Informatics | **Tools Used**: 4 | **Servers**: 2

## Description

Mine scientific literature: PubMed search, arXiv search, web search, and Tavily deep search.

## Tools Used

- **`pubmed_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`
- **`search_literature`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`search_web`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`tavily_search`** from `search-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search`

## Workflow

1. Search PubMed
2. Search arXiv
3. Web search
4. Tavily deep search

## Test Case

### Input
```json
{
    "query": "CRISPR Cas9 gene therapy 2024"
}
```

### Expected Steps
1. Search PubMed
2. Search arXiv
3. Web search
4. Tavily deep search

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
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
    sessions["search-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/7/Origene-Search", "streamable-http")
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")

    # Execute workflow steps
    # Step 1: Search PubMed
    result_1 = await sessions["search-server"].call_tool("pubmed_search", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Search arXiv
    result_2 = await sessions["server-1"].call_tool("search_literature", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Web search
    result_3 = await sessions["server-1"].call_tool("search_web", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Tavily deep search
    result_4 = await sessions["search-server"].call_tool("tavily_search", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
