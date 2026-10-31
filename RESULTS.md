# Results

## Test summary

| Suite | Tests | Status |
|---|---|---|
| Unit (all suites) | 147 | passed |
| Live (IGL, L1–L4) | 4 | passed |
| Live (Two Johns, JL1–JL4) | 4 | passed |
| Live (OpenAI, E8/E9) | 2 | passed |
| **Total live** | **10** | **passed** |

Live tests require a running IGL Memory server and valid user tokens.

## What was demonstrated

| Stage | Result |
|---|---|
| CE.1-r3 | Deterministic CognitiveFrame selection: HARD / SOFT / DEFAULT / UNDETERMINED, revision semantics, DecisionTrace |
| CE.2.1b | Adapter → Engine: memory in namespaced context, fail-closed on Gate unavailability, `allowed_memory_ref` in trace |
| CE.2.2 | Live IGL: record, correction, reject, forget → Gate → Adapter → Engine |
| CE.2.3 (mock) | Two Johns on mock memory: different memory → different frames |
| CE.2.3-live | Two Johns on real IGL: different users → different frames, same users → same frame, forget removes dependency, deterministic replay |
| CE.3.1 | PredicateRegistry (immutable, namespaced) |
| CE.3.2 | Predicate evaluation in Engine, fail-closed HARD |
| CE.3.3 | Natural-language frame selection (P6a-P6e) |
| CE.4.1 | Execution Layer structures (ExecutionRequest, ExecutionResult, ExecutionTrace, ResponseGateResult, ModelCatalog) |
| CE.4.2 | Frame-driven prompt builder + MockAdapter |
| CE.4.3 | Deterministic ResponseGate (NOT_VERIFIED when semantics unverified) |
| CE.4.4/4.5 | CognitiveAgent full cycle; MODEL_UNAVAILABLE; memory disclosure protection |
| CE.4.6 | OpenAI-compatible adapter via aitunnel, live E8/E9 |
| CE.5 | CLI `igl-cognitive respond` (mock + live), JSON trace |

Deterministic predicates can recognise predefined language patterns in user text and select a CognitiveFrame without a flat flag and without an LLM. This is pattern matching, not general NLU.

### Main experimental claim

> Given the same Cognitive Engine, the same operational context,
> and the same request, differences in the **allowed** long-term memory
> of IGL produced **reproducible** differences in the selected CognitiveFrame
> in the tested scenarios.

This is a narrow, testable claim. It is **not** a claim about reasoning quality,
consciousness, or superiority over existing agents.

### What was NOT demonstrated

- Improvement in reasoning quality.
- Superiority over a standard LLM agent.
- Self-acquisition of new strategies.
- Learning from consequences.
- Transfer of behavior across different LLMs.
- Subjective consciousness.
- Autonomous discovery of new modes of thinking.
- Memory-adaptive execution through the CLI (Engine-level only).
- Outbound disclosure policy for secrets pasted by the user.
- Semantic verification of LLM output.

**Important:** the memory-conditioned rules in the current experiments
are defined by the developer. The experiments demonstrate that **allowed memory
can change the selected frame**, not that the agent discovers new modes of
thinking on its own.

**Important:** `ResponseGate` does **not** verify factual or semantic correctness.
`PASS` means "all configured deterministic checks passed", not "the answer is true".

### Verified end-to-end on a live LLM

Command:

```bash
igl-cognitive respond \
  --text "Докажи математически, что 2+2=4" \
  --json \
  --api-key $OPENAI_API_KEY \
  --base-url https://api.aitunnel.ru/v1 \
  --model gpt-5.6-luna
  ```

Observed:

frame: verification / mathematics / formal_verification / strong;

model: gpt-5.6-luna via openai_compatible adapter;

response: a step-by-step proof in Peano arithmetic;

response_gate.status: NOT_VERIFIED (semantics not checked).

The mathematical proof returned by the LLM is not formally verified.
NOT_VERIFIED is the correct status here.

## Reproducibility

All unit tests are deterministic and require no external services.
Live tests require:
- running IGL Memory server (`IGL_BASE_URL`);
- user tokens (`IGL_USER_TOKEN_A`, `IGL_USER_TOKEN_B`);
- test database (do not run against production data).

`DecisionTrace` contains only hashes, fired rules, and frame metadata.
No plaintext user input, no tokens.

