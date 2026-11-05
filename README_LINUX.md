```markdown
# IGL Cognitive Runtime — Linux v0.2.0 (minimal)

This document describes the **Linux x86-64 binary** of v0.2.0.
It is intentionally minimal: no battery, no MCP.

## What is in the Linux binary

- **Cognitive Engine** — deterministic frame selection.
- **Execution Layer** — OpenAI-compatible adapter, ResponseGate.
- **CLI** — `igl-cognitive respond` (mock + real).
- **Trace store** — file-based JSONL, multi-user isolated.

## What is NOT in the Linux binary

- **Battery audit layer** (`igl_battery`) — requires PyBaMM.
- **MCP server** (`igl_mcp`) — requires `mcp` SDK.

For battery and MCP, use the Windows v0.2 build.

## Quick start

```bash
tar -xzf igl-cognitive-runtime-v0.2-linux-x86_64.tar.gz
chmod +x igl_cli_launcher.dist/igl-cognitive

# mock (no network)
./igl_cli_launcher.dist/igl-cognitive respond --text "prove mathematically that 2+2=4" --mock --json

# real LLM
export OPENAI_API_KEY=...
export OPENAI_BASE_URL=https://api.openai.com/v1
./igl_cli_launcher.dist/igl-cognitive respond --text "prove mathematically that 2+2=4" --json --model gpt-4o-mini
```

## Why Linux is minimal
The Linux binary is built without PyBaMM and MCP to keep it small and
fast to build. PyBaMM pulls scipy/pandas, which makes the binary
~300 MB and the build ~1.5 hours.

If you need battery or MCP on Linux, install from source:

```bash
pip install -e ".[dev,openai,mcp,battery]"
python -m igl_battery.pybamm_demo
python -m igl_mcp.server
```

## Limitations

ResponseGate does not verify semantic correctness.

## LLM output is non-deterministic.

Secrets in user text are not stripped.