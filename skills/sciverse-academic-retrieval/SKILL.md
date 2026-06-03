---
name: sciverse-academic-retrieval
description: Citation-grade academic literature retrieval (search, semantic chunks, byte-range read, figure fetch) over Sciverse, an open scientific platform indexing peer-reviewed and preprint papers.
license: Apache-2.0
metadata:
    skill-author: OpenDataLab
---

# Sciverse Academic Retrieval

Connects to the **Sciverse** SCP Server via the SCP Hub MCP gateway to perform
**citation-grade scientific literature retrieval** over a corpus that includes
peer-reviewed papers (Nature, Cell, …), preprints (arXiv, bioRxiv, …) and other
academic sources.

The server exposes 5 tools designed for **RAG by autonomous research agents**:
structured metadata search, natural-language semantic chunk retrieval,
byte-range source-text reading, and figure/table image fetching — all returning
stable `doc_id` / `chunk_id` for reproducible citation.

## Usage

```python
import asyncio
import json
import base64
from mcp.client.streamable_http import streamablehttp_client
from mcp import ClientSession


class SciverseClient:
    """Sciverse SCP Server client (5 academic-retrieval tools).

    All requests transparently proxied by the SCP Hub to the Sciverse backend.
    Authentication uses the SCP-HUB-API-KEY header (your SCP Platform key).
    """

    def __init__(self, server_url: str, api_key: str):
        self.server_url = server_url
        self.api_key = api_key
        self.session = None

    async def connect(self):
        try:
            self.transport = streamablehttp_client(
                url=self.server_url,
                headers={"SCP-HUB-API-KEY": self.api_key},
            )
            self.read, self.write, self.get_session_id = await self.transport.__aenter__()
            self.session_ctx = ClientSession(self.read, self.write)
            self.session = await self.session_ctx.__aenter__()
            await self.session.initialize()
            return True
        except Exception as e:
            print(f"[sciverse] connect failed: {e}")
            return False

    async def disconnect(self):
        if self.session:
            await self.session_ctx.__aexit__(None, None, None)
        if hasattr(self, "transport"):
            await self.transport.__aexit__(None, None, None)

    def parse_text_result(self, result):
        """Extract concatenated text from a tool result's content blocks.

        Works for: search_papers, semantic_search, read_content, list_catalog.
        Returns: str (the tool's JSON payload as text).
        """
        if isinstance(result, dict):
            content_list = result.get("content") or []
        else:
            content_list = getattr(result, "content", []) or []
        texts = []
        for item in content_list:
            if isinstance(item, dict):
                if item.get("type") == "text":
                    texts.append(item.get("text") or "")
            else:
                if getattr(item, "type", None) == "text":
                    texts.append(getattr(item, "text", "") or "")
        return "".join(texts)

    def parse_image_result(self, result):
        """Extract a figure/table image (used by get_resource).

        Returns: dict with keys 'mime_type' (e.g. 'image/png') and 'bytes'
                 (decoded binary). Returns None if the result is not an image.
        """
        if isinstance(result, dict):
            content_list = result.get("content") or []
        else:
            content_list = getattr(result, "content", []) or []
        for item in content_list:
            data = item.get("data") if isinstance(item, dict) else getattr(item, "data", None)
            mime = item.get("mimeType") if isinstance(item, dict) else getattr(item, "mimeType", None)
            type_ = item.get("type") if isinstance(item, dict) else getattr(item, "type", None)
            if type_ == "image" and data:
                return {"mime_type": mime, "bytes": base64.b64decode(data)}
        return None
```

### Initialize and use

```python
SERVER_URL = "https://scp.intern-ai.org.cn/api/v1/mcp/43/Sciverse"
API_KEY = "<YOUR_SCP_HUB_API_KEY>"


async def main():
    client = SciverseClient(SERVER_URL, API_KEY)
    if not await client.connect():
        print("connect failed")
        return
    try:
        # 1. Structured search: recent transformer papers
        result = await client.session.call_tool(
            "search_papers",
            arguments={
                "query": "transformer attention",   # BM25 over title/abstract/journal
                "year_from": 2023,
                "page_size": 5,
            },
        )
        papers = json.loads(client.parse_text_result(result))
        print(f"search_papers hits: {len(papers.get('hits', []))}")

        # 2. Semantic search: RAG-style chunk retrieval
        result = await client.session.call_tool(
            "semantic_search",
            arguments={"query": "How does transformer attention work?", "top_k": 3},
        )
        chunks = json.loads(client.parse_text_result(result))
        for hit in chunks.get("hits", []):
            print(f"  - {hit['title']} (score={hit['score']:.3f}, doc_id={hit['doc_id']})")

        # 3. Read content: expand context around a known offset
        if chunks.get("hits"):
            first = chunks["hits"][0]
            result = await client.session.call_tool(
                "read_content",
                arguments={"doc_id": first["doc_id"], "offset": first["offset"], "limit": 4096},
            )
            text_window = json.loads(client.parse_text_result(result))
            print(f"read_content next_offset={text_window.get('next_offset')} more={text_window.get('more')}")

        # 4. List catalog: discover available filter fields and operators
        result = await client.session.call_tool(
            "list_catalog", arguments={"include_sample_values": False},
        )
        catalog = json.loads(client.parse_text_result(result))
        print(f"available filter fields: {len(catalog.get('fields', []))}")

        # 5. Get resource: fetch a figure referenced inside read_content's Markdown
        # (Only call after read_content returned a Markdown snippet with ![alt](file_name).)
        # result = await client.session.call_tool(
        #     "get_resource", arguments={"file_name": "figures/fig-3.png"},
        # )
        # image = client.parse_image_result(result)
        # if image:
        #     from pathlib import Path
        #     Path("fig-3.png").write_bytes(image["bytes"])
    finally:
        await client.disconnect()


asyncio.run(main())
```

### Tool: `search_papers`

Structured metadata search by author, journal, year, subject, etc. Use when
the user knows specific filter values ("Hinton's papers from 2020-2023",
"Nature papers on CRISPR"). Do **not** use for free-text Q&A — that's
`semantic_search`.

- **Args**:
  - `query` (str, optional) — BM25 keyword over title/abstract/journal
  - `title_contains` (str, optional) — Substring match on title
  - `abstract_contains` (str, optional) — Substring match on abstract
  - `authors` (list[str], optional) — Any of these authors matches
  - `year_from` / `year_to` (int, optional) — Publication year range (inclusive)
  - `journals` (list[str], optional) — Journal names (any match)
  - `subjects` (list[str], optional) — Subject classification (e.g. "biology")
  - `sort_by_year` (str, default `"desc"`) — `desc` / `asc` / `none`
  - `page` (int, default 1), `page_size` (int, default 10, max 50)
  - `filters_advanced` (list, optional) — Escape hatch with full operator set
    (`FILTER_OP_EQ`, `IN`, `CONTAINS`, `GTE`, `LTE`, …) for fields not surfaced above
- **Returns**: JSON `{hits: [...], total: int}` where each hit has
  `doc_id`, `title`, `author`, `abstract`, `publication_venue_name`,
  `publication_published_year`.

### Tool: `semantic_search`

Natural-language semantic search returning relevant **paper chunks** for
RAG-style answering. Use for free-text questions ("How does attention
work?"). Typical chain: `semantic_search` → pick chunk → `read_content`.

- **Args**:
  - `query` (str, required) — Natural-language query, 1-200 words optimal
  - `top_k` (int, default 10, max 30)
  - `source_types` (list[str], optional) — Filter by `web` / `pdf`
  - `mode` (str, default `"balanced"`) — `fast` (~200ms keyword only) /
    `balanced` (~600ms hybrid) / `quality` (~2-4s LLM-rewrite + hybrid)
- **Returns**: JSON `{hits: [...]}` where each hit has
  `chunk_id`, `doc_id`, `chunk` (the matched text), `score`, `title`,
  `offset` (byte offset into source doc — pass to `read_content` for expansion).

### Tool: `read_content`

Read a UTF-8 byte range of a paper's source text. Typically called with a
`doc_id`/`offset` returned by `semantic_search` to expand context (read more
bytes before or after a chunk for fuller answers).

- **Args**:
  - `doc_id` (str, required) — Paper ID from `search_papers` / `semantic_search`
  - `offset` (int, default 0) — Byte offset to start reading
  - `limit` (int, default 4096, max 16384) — Bytes to read
- **Returns**: JSON `{text: str, bytes_returned: int, next_offset: int, more: bool}`.
  Markdown text may contain figure references like `![alt](file_name)` — pass
  `file_name` to `get_resource` to fetch the image.

### Tool: `get_resource`

Fetch the binary bytes of a paper figure / table image referenced inside
`read_content`'s Markdown. Use when the user asks to see / describe a figure
and `read_content` output contains an image reference.

- **Args**:
  - `file_name` (str, required) — Relative path from the Markdown `![alt](file_name)`.
    Must not contain `..` or start with `/`.
- **Returns**: Image content block — `data` (base64) + `mimeType` (`image/*`).
  Multimodal agents (Claude, GPT-4V, Gemini, …) can read it directly.

### Tool: `list_catalog`

Returns the schema catalog for `search_papers`: every field name, type,
whether it's filterable / sortable / default-returned, human description, and
applicable filter operators. Use when constructing precise `search_papers`
filters or facing an ambiguous field need.

- **Args**:
  - `include_sample_values` (bool, default `false`) — If `true`, also fetch
    top-20 values for enum-like fields (24h cached, ~100s of ms first call).
- **Returns**: JSON `{fields: [...]}` where each field has `name`, `type`
  (`string`/`integer`/`list[string]`/…), `filterable`, `sortable`,
  `default_return`, `description`, `applicable_operators`, and optionally
  `sample_values`.

### Use Cases

- **Drug discovery / pharmacology**: literature scoping for a target before
  triggering wet-lab skills; RAG context for ADMET / MoA reasoning.
- **Protein science**: gather structure/function papers around a UniProt ID
  before predicting mutations or binding sites.
- **Genomics & rare disease**: pull recent papers on a variant / phenotype
  for evidence-grade reasoning, then cite by `doc_id`.
- **Chemistry / materials**: find prior art around a SMILES or reaction
  before computing properties.
- **Cross-domain literature review**: agentic survey writing — chain
  `semantic_search` → `read_content` to assemble citation-grounded
  summaries with stable `doc_id` references for verifiability.
