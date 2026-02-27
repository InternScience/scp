---
name: drug_metabolism_study
description: "Drug Metabolism Study - Study drug metabolism: FDA metabolism data, ChEMBL metabolism records, PubChem compound data, and clinical pharmacology. Use this skill for drug metabolism tasks involving get metabolism by id get pharmacokinetics by drug name get compound by name get clinical pharmacology by drug name. Combines 4 tools from 3 SCP server(s)."
---

# Drug Metabolism Study

**Discipline**: Drug Metabolism | **Tools Used**: 4 | **Servers**: 3

## Description

Study drug metabolism: FDA metabolism data, ChEMBL metabolism records, PubChem compound data, and clinical pharmacology.

## Tools Used

- **`get_metabolism_by_id`** from `chembl-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/4/Origene-ChEMBL`
- **`get_pharmacokinetics_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_compound_by_name`** from `pubchem-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/8/Origene-PubChem`
- **`get_clinical_pharmacology_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`

## Workflow

1. Get ChEMBL metabolism data
2. Get FDA pharmacokinetics
3. Get PubChem compound data
4. Get clinical pharmacology

## Test Case

### Input
```json
{
    "drug_name": "warfarin",
    "met_id": 1
}
```

### Expected Steps
1. Get ChEMBL metabolism data
2. Get FDA pharmacokinetics
3. Get PubChem compound data
4. Get clinical pharmacology

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
    # Step 1: Get ChEMBL metabolism data
    result_1 = await sessions["chembl-server"].call_tool("get_metabolism_by_id", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get FDA pharmacokinetics
    result_2 = await sessions["fda-drug-server"].call_tool("get_pharmacokinetics_by_drug_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get PubChem compound data
    result_3 = await sessions["pubchem-server"].call_tool("get_compound_by_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Get clinical pharmacology
    result_4 = await sessions["fda-drug-server"].call_tool("get_clinical_pharmacology_by_drug_name", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
