# IGL Cognitive Runtime — Demo

## Download

Download the Linux x86-64 binary from
[GitHub Releases](https://github.com/IGLANGUAGE/igl-cognitive-runtime/releases).

```bash
tar -xzf igl-cognitive-runtime-v0.1.0-linux-x86_64.tar.gz
chmod +x igl_cli_launcher.dist/igl-cognitive
```

## Demo A: Mock (no network, no key)

```bash
./igl_cli_launcher.dist/igl-cognitive respond \
  --text "докажи математически" \
  --mock \
  --json
```
Returns a JSON trace with frame_selection, mock-strong, NOT_VERIFIED.


## Demo B: Real LLM (any OpenAI-compatible endpoint)

### Option 1: Ollama (local, free, offline)

```bash
export OPENAI_BASE_URL=http://localhost:11434/v1
export OPENAI_API_KEY=ollama

./igl_cli_launcher.dist/igl-cognitive respond \
  --text "Prove mathematically that 2+2=4" \
  --json \
  --model llama3
```

### Option 2: OpenAI

```bash
export OPENAI_BASE_URL=https://api.openai.com/v1
export OPENAI_API_KEY=sk-...

./igl_cli_launcher.dist/igl-cognitive respond \
  --text "Prove mathematically that 2+2=4" \
  --json \
  --model gpt-4o-mini
```

### Option 3: OpenRouter

```bash
export OPENAI_BASE_URL=https://openrouter.ai/api/v1
export OPENAI_API_KEY=sk-or-...

./igl_cli_launcher.dist/igl-cognitive respond \
  --text "Prove mathematically that 2+2=4" \
  --json \
  --model anthropic/claude-3.5-sonnet
```

## Observed
Register: verification

Coordinate system: mathematics

Strategy: formal_verification

Model tier: strong

Response Gate: NOT_VERIFIED (semantics not checked)

The LLM-generated proof is not formally verified.