---
name: protein_function_annotation
description: "Protein Function Annotation Pipeline - Annotate protein function: UniProt metadata, InterPro domains, functional prediction, and GO enrichment. Use this skill for proteomics tasks involving query uniprot query interpro predict protein function get functional enrichment. Combines 4 tools from 2 SCP server(s)."
---

# Protein Function Annotation Pipeline

**Discipline**: Proteomics | **Tools Used**: 4 | **Servers**: 2

## Description

Annotate protein function: UniProt metadata, InterPro domains, functional prediction, and GO enrichment.

## Tools Used

- **`query_uniprot`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`query_interpro`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`predict_protein_function`** from `server-1` (sse) - `https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory`
- **`get_functional_enrichment`** from `string-server` (streamable-http) - `https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING`

## Workflow

1. Get UniProt metadata
2. Get InterPro domain annotations
3. Predict protein function
4. Run GO enrichment analysis

## Test Case

### Input
```json
{
    "uniprot_id": "P04637"
}
```

### Expected Steps
1. Get UniProt metadata
2. Get InterPro domain annotations
3. Predict protein function
4. Run GO enrichment analysis

## Usage Example

> **Note:** Replace `<YOUR_SCP_HUB_API_KEY>` with your own SCP Hub API Key. You can obtain one from the [SCP Platform](https://scp.intern-ai.org.cn).

```python
import asyncio
import json
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client
from mcp.client.sse import sse_client

SERVERS = {
    "server-1": "https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory",
    "string-server": "https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING"
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
    sessions["server-1"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/1/VenusFactory", "sse")
    sessions["string-server"], _, _ = await connect("https://scp.intern-ai.org.cn/api/v1/mcp/6/Origene-STRING", "streamable-http")

    # Execute workflow steps
    # Step 1: Get UniProt metadata
    result_1 = await sessions["server-1"].call_tool("query_uniprot", arguments={})
    data_1 = parse(result_1)
    print(f"Step 1 result: {json.dumps(data_1, indent=2, ensure_ascii=False)[:500]}")

    # Step 2: Get InterPro domain annotations
    result_2 = await sessions["server-1"].call_tool("query_interpro", arguments={})
    data_2 = parse(result_2)
    print(f"Step 2 result: {json.dumps(data_2, indent=2, ensure_ascii=False)[:500]}")

    # Step 3: Predict protein function
    result_3 = await sessions["server-1"].call_tool("predict_protein_function", arguments={})
    data_3 = parse(result_3)
    print(f"Step 3 result: {json.dumps(data_3, indent=2, ensure_ascii=False)[:500]}")

    # Step 4: Run GO enrichment analysis
    result_4 = await sessions["string-server"].call_tool("get_functional_enrichment", arguments={})
    data_4 = parse(result_4)
    print(f"Step 4 result: {json.dumps(data_4, indent=2, ensure_ascii=False)[:500]}")

    # Cleanup
    print("Workflow complete!")

if __name__ == "__main__":
    asyncio.run(main())
```
