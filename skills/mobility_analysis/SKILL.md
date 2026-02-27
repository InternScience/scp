---
name: mobility_analysis
description: "Charge Carrier Mobility Analysis - Analyze carrier mobility: calculate new mobility, compute vacuum permittivity, and error analysis. Use this skill for semiconductor physics tasks involving calculate new mobility calculate vacuum permittivity calculate absolute error calculate mean square. Combines 4 tools from 2 SCP server(s)."
---

# Charge Carrier Mobility Analysis

**Discipline**: Semiconductor Physics | **Tools Used**: 4 | **Servers**: 2

## Description

Analyze carrier mobility: calculate new mobility, compute vacuum permittivity, and error analysis.

## Tools Used

- **`calculate_new_mobility`** from `server-21` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations`
- **`calculate_vacuum_permittivity`** from `server-21` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations`
- **`calculate_absolute_error`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`
- **`calculate_mean_square`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`

## Workflow

1. Calculate new mobility
2. Compute vacuum permittivity
3. Calculate measurement error
4. Compute mean square statistics

## Test Case

### Input
```json
{
    "mobility_data": [
        1500,
        1450,
        1520
    ]
}
```

### Expected Steps
1. Calculate new mobility
2. Compute vacuum permittivity
3. Calculate measurement error
4. Compute mean square statistics

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-21": "https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations",
    "server-26": "https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis"
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
    sessions["server-21"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations", "streamable-http")
    sessions["server-26"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis", "streamable-http")

    # Execute workflow steps
    # Step 1: Calculate new mobility
    result_1 = await sessions["server-21"].call_tool("calculate_new_mobility", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Compute vacuum permittivity
    result_2 = await sessions["server-21"].call_tool("calculate_vacuum_permittivity", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Calculate measurement error
    result_3 = await sessions["server-26"].call_tool("calculate_absolute_error", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Compute mean square statistics
    result_4 = await sessions["server-26"].call_tool("calculate_mean_square", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
