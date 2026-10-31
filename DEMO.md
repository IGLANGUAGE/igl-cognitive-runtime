# IGL Cognitive Runtime — Demo

## Download

Download the Linux x86-64 binary from GitHub Releases.

## Demo: Context-Aware AI Execution

```bash
export OPENAI_API_KEY=...
export OPENAI_BASE_URL=https://api.aitunnel.ru/v1

./igl-cognitive respond \
  --text "Prove mathematically that 2+2=4" \
  --json \
  --model gpt-5.6-luna