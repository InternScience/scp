---
name: electrical_circuit_analysis
description: "Electrical Circuit Analysis - Analyze electrical circuit: compute capacitance, convert resistance units, calculate total charge, and duty cycle. Use this skill for electrical engineering tasks involving convert resistance kOhm to Ohm calculate geometric term calculate absolute error. Combines 3 tools from 3 SCP server(s)."
---

# Electrical Circuit Analysis

**Discipline**: Electrical Engineering | **Tools Used**: 3 | **Servers**: 3

## Description

Analyze electrical circuit: compute capacitance, convert resistance units, calculate total charge, and duty cycle.

## Tools Used

- **`convert_resistance_kOhm_to_Ohm`** from `server-21` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations`
- **`calculate_geometric_term`** from `server-20` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/20/Materials_Mechanics_and_Fracture_Analysis`
- **`calculate_absolute_error`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`

## Workflow

1. Convert resistance kOhm to Ohm
2. Calculate geometric parameters
3. Compute measurement error

## Test Case

### Input
```json
{
    "resistance_kohm": 4.7,
    "measured": 14.7,
    "true_val": 15.0
}
```

### Expected Steps
1. Convert resistance kOhm to Ohm
2. Calculate geometric parameters
3. Compute measurement error

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
    "server-20": "https://scp.intern-ai.org.cn/api/v1/mcp/20/Materials_Mechanics_and_Fracture_Analysis",
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
    sessions["server-20"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/20/Materials_Mechanics_and_Fracture_Analysis", "streamable-http")
    sessions["server-26"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis", "streamable-http")

    # Execute workflow steps
    # Step 1: Convert resistance kOhm to Ohm
    result_1 = await sessions["server-21"].call_tool("convert_resistance_kOhm_to_Ohm", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Calculate geometric parameters
    result_2 = await sessions["server-20"].call_tool("calculate_geometric_term", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Compute measurement error
    result_3 = await sessions["server-26"].call_tool("calculate_absolute_error", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
