---
name: capacitance-calculation
description: Calculate electrical capacitance from geometric parameters and dielectric properties for circuit design.
license: MIT license
metadata:
    skill-author: PJLab
---

# Electrical Capacitance Calculation

## Usage

```python
import asyncio
import json
from mcp.client.streamable_http import streamablehttp_client
from mcp import ClientSession

class ElectricalClient:
    def __init__(self, server_url: str, api_key: str):
        self.server_url = server_url
        self.api_key = api_key
        self.session = None

    async def connect(self):
        try:
            self.transport = streamablehttp_client(url=self.server_url, headers={"SCP-HUB-API-KEY": self.api_key})
            self.read, self.write, self.get_session_id = await self.transport.__aenter__()
            self.session_ctx = ClientSession(self.read, self.write)
            self.session = await self.session_ctx.__aenter__()
            await self.session.initialize()
            return True
        except:
            return False

    async def disconnect(self):
        if self.session:
            await self.session_ctx.__aexit__(None, None, None)
        if hasattr(self, 'transport'):
            await self.transport.__aexit__(None, None, None)

    def parse_result(self, result):
        try:
            if hasattr(result, 'content') and result.content:
                return json.loads(result.content[0].text)
            return str(result)
        except:
            return {"error": "parse error"}

## Initialize and use
client = ElectricalClient("https://scp.intern-ai.org.cn/api/v1/mcp/21/Electrical_Engineering_and_Circuit_Calculations", "<your-api-key>")
await client.connect()

# Calculate parallel plate capacitance
epsilon_0 = 8.854e-12  # F/m (vacuum permittivity)
epsilon_r = 4.0  # Relative permittivity
area = 0.01  # m² (plate area)
distance = 0.001  # m (separation)
capacitance = epsilon_0 * epsilon_r * area / distance
print(f"Capacitance: {capacitance*1e12:.2f} pF")

await client.disconnect()
```

### Use Cases
- Circuit design, capacitor selection, electromagnetic simulations, filter design
