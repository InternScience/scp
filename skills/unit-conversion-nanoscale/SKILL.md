---
name: unit-conversion-nanoscale
description: Convert physical quantities and units at nanoscale for materials science and nanotechnology applications.
license: MIT license
metadata:
    skill-author: PJLab
---

# Unit Conversion for Nanoscale

## Usage

```python
import asyncio
import json
from mcp.client.streamable_http import streamablehttp_client
from mcp import ClientSession

class UnitClient:
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
client = UnitClient("https://scp.intern-ai.org.cn/api/v1/mcp/27/Physical_Quantities_Conversion", "<your-api-key>")
await client.connect()

# Nanoscale unit conversions
nm_to_m = 1e-9
angstrom_to_m = 1e-10

length_nm = 50  # nanometers
length_m = length_nm * nm_to_m
length_angstrom = length_m / angstrom_to_m

print(f"{length_nm} nm = {length_m:.2e} m = {length_angstrom:.1f} Å")

# Energy conversion (eV to Joules)
eV_to_J = 1.602e-19
energy_eV = 2.5
energy_J = energy_eV * eV_to_J
print(f"{energy_eV} eV = {energy_J:.2e} J")

await client.disconnect()
```

### Use Cases
- Nanotechnology, semiconductor physics, quantum mechanics, materials characterization
