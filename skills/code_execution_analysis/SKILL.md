---
name: code_execution_analysis
description: "Computational Analysis via Code Execution - Execute custom computational analysis code, analyze software, and search for reference implementations. Use this skill for computational science tasks involving exec code software analysis search dataset search literature. Combines 4 tools from 2 SCP server(s)."
---

# Computational Analysis via Code Execution

**Discipline**: Computational Science | **Tools Used**: 4 | **Servers**: 2

## Description

Execute custom computational analysis code, analyze software, and search for reference implementations.

## Tools Used

- **`exec_code`** from `server-18` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/18/Thoth-OP`
- **`software_analysis`** from `server-18` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/18/Thoth-OP`
- **`search_dataset`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`search_literature`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`

## Workflow

1. Execute analysis code
2. Analyze software requirements
3. Search for datasets
4. Search for methods literature

## Test Case

### Input
```json
{
    "code": "print('hello')",
    "query": "machine learning protein prediction"
}
```

### Expected Steps
1. Execute analysis code
2. Analyze software requirements
3. Search for datasets
4. Search for methods literature

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-18": "https://scp.intern-ai.org.cn/api/v1/mcp/18/Thoth-OP",
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
    sessions["server-18"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/18/Thoth-OP", "streamable-http")
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")

    # Execute workflow steps
    # Step 1: Execute analysis code
    result_1 = await sessions["server-18"].call_tool("exec_code", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Analyze software requirements
    result_2 = await sessions["server-18"].call_tool("software_analysis", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Search for datasets
    result_3 = await sessions["server-1"].call_tool("search_dataset", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Search for methods literature
    result_4 = await sessions["server-1"].call_tool("search_literature", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
