---
name: atc_drug_classification
description: "ATC Drug Classification Lookup - Look up drug in ATC classification: ChEMBL ATC class, FDA drug info, PubChem compound, and mechanism of action. Use this skill for pharmacology tasks involving get atc class by level5 get mechanism of action by drug name get compound by name get drug by name. Combines 4 tools from 3 SCP server(s)."
---

# ATC Drug Classification Lookup

**Discipline**: Pharmacology | **Tools Used**: 4 | **Servers**: 3

## Description

Look up drug in ATC classification: ChEMBL ATC class, FDA drug info, PubChem compound, and mechanism of action.

## Tools Used

- **`get_atc_class_by_level5`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_mechanism_of_action_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_compound_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`get_drug_by_name`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`

## Workflow

1. Get ATC classification
2. Get mechanism of action
3. Get PubChem compound
4. Get ChEMBL drug data

## Test Case

### Input
```json
{
    "atc_code": "C10AA05",
    "drug_name": "atorvastatin"
}
```

### Expected Steps
1. Get ATC classification
2. Get mechanism of action
3. Get PubChem compound
4. Get ChEMBL drug data

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "chembl-server": "https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL",
    "fda-drug-server": "https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug",
    "pubchem-server": "https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem"
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
    sessions["chembl-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL", "streamable-http")
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["pubchem-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem", "streamable-http")

    # Execute workflow steps
    # Step 1: Get ATC classification
    result_1 = await sessions["chembl-server"].call_tool("get_atc_class_by_level5", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get mechanism of action
    result_2 = await sessions["fda-drug-server"].call_tool("get_mechanism_of_action_by_drug_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get PubChem compound
    result_3 = await sessions["pubchem-server"].call_tool("get_compound_by_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get ChEMBL drug data
    result_4 = await sessions["chembl-server"].call_tool("get_drug_by_name", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
