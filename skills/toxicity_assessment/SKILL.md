---
name: toxicity_assessment
description: "Drug Toxicity Assessment - Comprehensive toxicity assessment: FDA adverse reactions, nonclinical toxicology, carcinogenicity data, and ADMET prediction. Use this skill for toxicology tasks involving get adverse reactions by drug name get nonclinical toxicology info by drug name get carcinogenic mutagenic fertility impairment info by drug name pred molecule admet. Combines 4 tools from 2 SCP server(s)."
---

# Drug Toxicity Assessment

**Discipline**: Toxicology | **Tools Used**: 4 | **Servers**: 2

## Description

Comprehensive toxicity assessment: FDA adverse reactions, nonclinical toxicology, carcinogenicity data, and ADMET prediction.

## Tools Used

- **`get_adverse_reactions_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_nonclinical_toxicology_info_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`get_carcinogenic_mutagenic_fertility_impairment_info_by_drug_name`** from `fda-drug-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug`
- **`pred_molecule_admet`** from `server-3` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model`

## Workflow

1. Get FDA adverse reactions
2. Get nonclinical toxicology
3. Get carcinogenicity info
4. Predict ADMET toxicity endpoints

## Test Case

### Input
```json
{
    "drug_name": "acetaminophen",
    "smiles": "CC(=O)Nc1ccc(O)cc1"
}
```

### Expected Steps
1. Get FDA adverse reactions
2. Get nonclinical toxicology
3. Get carcinogenicity info
4. Predict ADMET toxicity endpoints

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "fda-drug-server": "https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug",
    "server-3": "https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model"
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
    sessions["fda-drug-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/14/Origene-FDADrug", "streamable-http")
    sessions["server-3"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/3/DrugSDA-Model", "streamable-http")

    # Execute workflow steps
    # Step 1: Get FDA adverse reactions
    result_1 = await sessions["fda-drug-server"].call_tool("get_adverse_reactions_by_drug_name", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get nonclinical toxicology
    result_2 = await sessions["fda-drug-server"].call_tool("get_nonclinical_toxicology_info_by_drug_name", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Get carcinogenicity info
    result_3 = await sessions["fda-drug-server"].call_tool("get_carcinogenic_mutagenic_fertility_impairment_info_by_drug_name", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Predict ADMET toxicity endpoints
    result_4 = await sessions["server-3"].call_tool("pred_molecule_admet", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
