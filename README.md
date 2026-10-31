# IGL Cognitive Runtime

Experimental architecture for context-dependent selection of a reasoning mode.

## What this is

A context-aware execution layer for AI agents. It selects a **CognitiveFrame** —
a consistent configuration of register, coordinate system, reasoning strategy,
and model tier — **before** any LLM reasoning happens, then executes that frame
through an external OpenAI-compatible model and returns a structured trace.

Two things are separate:

- **Decision**: deterministic frame selection (Engine).
- **Execution**: the selected frame is executed by an external LLM (Adapter).

The frame selection is:

- deterministic (same inputs → same frame, same trace);
- independent of the executing LLM;
- conditioned on **allowed** long-term memory only (Engine level; see Limitations);
- fully traceable through `DecisionTrace` and `ExecutionTrace`.

## Components

- **CognitiveFrame** — a consistent configuration:
  `(register, coordinate_system, strategy, model_tier, revision)`.
- **Cognitive Engine** — deterministic frame selector.
  HARD rules constrain the allowed set.
  SOFT rules rank it.
  A classifier may propose a frame inside the allowed set.
  Default and explicit `UNDETERMINED` handle the rest.
- **Memory Gate Adapter** — receives only what IGL Gate allowed
  (`accepted`), keeps `withheld` and `constraints` in diagnostics,
  and produces a namespaced `memory_context` for the Engine.
- **DecisionTrace** — structured event: fired rules, classifier proposal,
  conflict, resolution, previous/selected frame, revision, hashes.
  No plaintext user input, no secrets.
- **PredicateRegistry** — immutable registry of deterministic predicates.
- **ExecutionRequestBuilder** — builds a minimal `ExecutionRequest`
  (system_prompt, user_prompt, model_id, temperature, max_tokens).
  No memory_context, no withheld, no constraints.
- - **ExecutionAdapter** — OpenAI-compatible adapter.
  Works with any OpenAI-compatible endpoint (OpenAI, OpenRouter, Groq,
  Together AI, local Ollama, and others). See "LLM backends" below.
  Receives only `ExecutionRequest`.
- **ResponseGate** — deterministic checks only.
  `PASS` means "all configured checks passed", **not** "factually true".
- **ExecutionTrace** — execution-side trace: model_id, prompt_hash, response_hash,
  response_gate, execution_status, model_catalog_hash.

## What this is not

- Not an LLM.
- Not a memory store. IGL Memory is a separate system.
- Not a replacement for IGL Memory.
- Not a claim of consciousness.
- Not a fact-checker. ResponseGate does not verify semantic correctness.

## Status

| Stage | Result |
|---|---|
| CE.1-r3 | Deterministic frame selection (HARD / SOFT / DEFAULT / UNDETERMINED) |
| CE.2.1b | Adapter → Engine, namespaced memory, fail-closed |
| CE.2.2 | Live IGL: correction / reject / forget → Gate → Engine |
| CE.2.3 (mock) | Different memory → different frames (mock) |
| CE.2.3-live | Two real IGL users → different frames (JL1–JL4) |
| CE.3 | Predicate evaluation, natural-language frame selection |
| CE.4.1 | Execution Layer contract structures (S1–S6) |
| CE.4.2 | Frame-driven execution builder + mock adapter |
| CE.4.3 | Deterministic response gate (NOT_VERIFIED) |
| CE.4.4/4.5 | CognitiveAgent full cycle, MODEL_UNAVAILABLE, memory disclosure protection |
| CE.4.6 | OpenAI-compatible adapter (any endpoint) |
| CE.5 | CLI: `igl-cognitive respond` (mock + live) |

Tests: **147 unit passed, 10 live passed** (requires a running IGL server
and `IGL_USER_TOKEN_A` / `IGL_USER_TOKEN_B`).

See `CONTRACT.md` for the full contract and `RESULTS.md` for what was
and was not demonstrated.

## LLM backends

Any OpenAI-compatible endpoint works. Set `OPENAI_API_KEY` and `OPENAI_BASE_URL`.

| Backend | Base URL | Notes |
|---|---|---|
| OpenAI | `https://api.openai.com/v1` | Standard models |
| OpenRouter | `https://openrouter.ai/api/v1` | Many models, one key |
| Groq | `https://api.groq.com/openai/v1` | Fast inference |
| Together AI | `https://api.together.xyz/v1` | Open models |
| Ollama (local) | `http://localhost:11434/v1` | Free, offline, no key |
| aitunnel | `https://api.aitunnel.ru/v1` | RU-region provider |

For local testing without any API key:

```bash
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_API_KEY=ollama
./igl-cognitive respond --text "prove mathematically that 2+2=4" --json --model llama3
```

## CLI

```bash
# Any OpenAI-compatible endpoint
export OPENAI_API_KEY=...
export OPENAI_BASE_URL=https://api.openai.com/v1   # or OpenRouter, Groq, Ollama, etc.
igl-cognitive respond --text "..." --json --model <model-id>
```

## Limitations

- **CLI does not connect to IGL Memory Gate yet.** Memory-conditioned frame
  selection is verified at the Engine level (live tests JL1–JL4), but the CLI
  does not pass `memory_context` or `allowed_memory_ref` into the full execution
  cycle. Do not describe the CLI as "memory-adaptive" until this is connected.
- **ResponseGate does not verify semantic correctness.**
  `NOT_VERIFIED` is the normal status when deterministic checks pass but
  semantics were not checked. `PASS` does not mean "factually true".
- **LLM output is non-deterministic.** `ExecutionTrace` records the actual
  response hash, not a guarantee of reproducibility.
- **Secrets in user text are not stripped.** If the user pastes a token into
  the request text, it will be forwarded to the LLM. Outbound policy is not
  implemented yet.

## Requirements

- Python ≥ 3.11
- For live tests: running IGL Memory server + user tokens

## Install

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e ".[dev]"
```

## Download

Download the Linux x86-64 binary from
[GitHub Releases](https://github.com/IGLANGUAGE/igl-cognitive-runtime/releases).

## Run tests

Unit only (no server):
```bash
pytest -q
```

Live (requires IGL server, tokens in env):
```bash
$env:IGL_BASE_URL="http://127.0.0.1:8000"
$env:IGL_USER_TOKEN_A="..."
$env:IGL_USER_TOKEN_B="..."
pytest -q -m live
```

## License

IGL Cognitive Runtime Community License. Free for non-commercial use
(personal, research, education). Commercial use requires a separate
written license. See LICENSE.
