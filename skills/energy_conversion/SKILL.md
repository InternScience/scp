---
name: energy_conversion
description: "Energy Unit Conversion Pipeline - Convert between energy units and analyze: MeV to Joules, scientific notation, and error calculation. Use this skill for physics tasks involving convert energy MeV to J convert to scientific notation format scientific notation calculate absolute error. Combines 4 tools from 2 SCP server(s)."
---

# Energy Unit Conversion Pipeline

**Discipline**: Physics | **Tools Used**: 4 | **Servers**: 2

## Description

Convert between energy units and analyze: MeV to Joules, scientific notation, and error calculation.

## Tools Used

- **`convert_energy_MeV_to_J`** from `server-22` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/22/Thermal_Fluid_Dynamics`
- **`convert_to_scientific_notation`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`
- **`format_scientific_notation`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`
- **`calculate_absolute_error`** from `server-26` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis`

## Workflow

1. Convert MeV to Joules
2. Convert to scientific notation
3. Format result
4. Calculate conversion error

## Test Case

### Input
```json
{
    "energy_MeV": 13.6,
    "expected_J": 2.18e-12
}
```

### Expected Steps
1. Convert MeV to Joules
2. Convert to scientific notation
3. Format result
4. Calculate conversion error

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-22": "https://scp.intern-ai.org.cn/api/v1/mcp/22/Thermal_Fluid_Dynamics",
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
    sessions["server-22"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/22/Thermal_Fluid_Dynamics", "streamable-http")
    sessions["server-26"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/26/Data_processing_and_statistical_analysis", "streamable-http")

    # Execute workflow steps
    # Step 1: Convert MeV to Joules
    result_1 = await sessions["server-22"].call_tool("convert_energy_MeV_to_J", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Convert to scientific notation
    result_2 = await sessions["server-26"].call_tool("convert_to_scientific_notation", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Format result
    result_3 = await sessions["server-26"].call_tool("format_scientific_notation", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Calculate conversion error
    result_4 = await sessions["server-26"].call_tool("calculate_absolute_error", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
