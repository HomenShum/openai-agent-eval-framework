# OpenAI Evaluation Backend

Reference TypeScript route handlers and a Jest evaluation harness for prototyping agent workflows. The tests supply substitutes for dependencies from the original host application. This checkout is not a standalone server and uses deterministic test identities rather than production authentication.

## 🚀 What This Does

The source provides reference examples for:
- **Agentic Workflows** - Multi-tool orchestration with streaming responses
- **JSON Extraction** - Structured output with schema enforcement
- **Embeddings** - Vector generation with built-in retry logic
- **RAG Pipelines** - Context gathering and synthesis patterns
- **Evaluation Framework** - Deterministic testing and performance metrics

## 🎯 Key Features

- **Route handler examples** - Adapt handlers, dependencies and authentication in your host application
- ✅ **Streaming SSE Support** - Real-time agent execution traces
- **Mocked route tests** - SDK substitutes exercise handler behavior without real provider credentials
- **Toy evaluation fixtures** - Source-defined metrics for local experiments; current results require a fresh run
- ✅ **Multi-User Simulation** - Test different user contexts easily

## 📦 Installation

### Prerequisites

- Node.js satisfying the declared package engine, `>=18.17`.
- npm; this repository includes a package-lock.json.
- A real OpenAI key is optional for intentionally selected provider-backed checks, not the default mocked route tests.

### Clone and install

```bash
git clone https://github.com/HomenShum/openai-agent-eval-framework
cd openai-agent-eval-framework
npm ci
```

The npm package is named `openai-eval-backend`; that is different from the directory Git creates above. Installation and current test results must be established in your own environment.

### Select the mocked route tests

Keep `OPENAI_API_KEY` unset when Jest reads its configuration. Choose the example for your shell:

```bash
# Bash
unset OPENAI_API_KEY
npm test -- --runInBand
```

```powershell
# PowerShell
Remove-Item Env:OPENAI_API_KEY -ErrorAction SilentlyContinue
npm test -- --runInBand
```

With no key, the current Jest configuration excludes the entire `openai/__tests__/` subtree. The adjacent route tests set their own dummy key and mock the SDK. This is a subset of the repository tests, not a complete live evaluation.

### Environment configuration

There is no committed `.env.example`, and the package scripts and Jest configuration do not load dotenv files. No environment file is required for the mocked selection above. For provider-backed checks, configure process variables in the terminal you intend to use, then select an exact test as described below. Merely creating a `.env` file does not implement loading.

## 🏗️ Project Structure

```
openai-agent-eval-framework/
├── openai/                    # Core route handlers
│   ├── agent/                 # Multi-tool agent orchestration
│   ├── ask/                   # Simple chat completions
│   ├── ask-json/              # Structured JSON outputs
│   ├── ask-mode/              # Preset response modes
│   ├── embeddings/            # Vector embeddings
│   ├── gather-context/        # RAG context collection
│   ├── organization-mode/     # Hierarchical note generation
│   └── __tests__/            # Test suites and evaluations
├── Interview/                 # Demo materials
│   ├── sse_samples/          # Captured SSE traces
│   ├── dataset/              # Sample evaluation data
│   └── *.md                  # Documentation
├── lib/                      # Shared utilities
│   └── eval/                 # Evaluation framework
├── scripts/                  # CLI utilities
└── test/__mocks__/          # Test mocks
```

## 🔧 Usage

### Integration with a host application

The handlers import host-framework and application modules. Jest maps `next/server`, `@/...`, Pinecone, Pusher and UUID imports to local substitutes in [jest.config.ts](jest.config.ts). Those test mappings are not a deployed backend.

To integrate a handler, adapt its imports and dependencies to your host layout, provide real identity/ownership checks, and mount it at your chosen route. The package has no `dev`, `start` or server-build script. The paths below describe reference handlers; this repository alone does not serve these URLs.

### Available Endpoints

| Endpoint | Purpose | Best For |
|----------|---------|----------|
| `/agent` | Multi-step workflows with tools | Complex tasks, automation |
| `/ask` | Simple text generation | Chat, Q&A |
| `/ask-json` | Structured JSON output | Forms, data extraction |
| `/ask-mode` | Preset response styles | Drafting, critiques |
| `/embeddings` | Vector generation | Semantic search, RAG |
| `/gather-context` | Context ranking | Knowledge retrieval |
| `/organization-mode` | Hierarchical outlines | Note-taking, documentation |

### Example: Agent Workflow

This request shape assumes your host application has mounted the agent handler at the illustrated URL.

```javascript
// Request
const response = await fetch('/api/llm/openai/agent', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-test-user-id': 'user-123' // Optional user context
  },
  body: JSON.stringify({
    messages: [
      { role: 'user', content: 'Create a project plan for AGI Safety' }
    ]
  })
});

// Stream SSE responses
const reader = response.body.getReader();
// ... process streaming chunks
```

## 🧪 Testing & Evaluation

### Test selection

[Jest configuration](jest.config.ts) reads key presence before test discovery:

| Process configuration | Discovery and behavior |
|-----------------------|------------------------|
| `OPENAI_API_KEY` absent | All of `openai/__tests__/` is excluded, including mini-eval and SSE-capture files there; adjacent mocked route tests remain selected. |
| `OPENAI_API_KEY` present | That directory becomes eligible too. Existing mocked tests remain mocked; selected live files have their own guards and may call provider APIs. |

Key presence does not turn every test into a live test. Known agent scenarios may also use preflight behavior automatically under Jest via `JEST_WORKER_ID` in [agent helpers](openai/agent/helpers.ts).

### Running a specific mocked test

With the key unset as in setup:

```bash
npm test -- --runInBand openai/agent/e2e.agent.graph-actions.test.ts
```

### Intentionally selected provider-backed checks

After configuring `OPENAI_API_KEY` in the current terminal, this exact file contains SDK/route smoke checks:

```bash
npm test -- --runInBand openai/__tests__/live.smoke.test.ts
```

The base smoke file enables its provider checks from key presence alone. Omitting `OPENAI_REAL_TESTS`, or setting it to `0`, does not disable that file. A broad `npm test` in a keyed terminal can therefore include provider calls.

Inspect the selected suite's guards before running it. Only the saved-log evaluator shown next additionally requires a nonempty `OPENAI_REAL_TESTS` value and an existing smoke log. For example, in Bash, with the key already configured:

```bash
OPENAI_REAL_TESTS=1 npm test -- --runInBand openai/__tests__/live.smoke.eval.test.ts
```

This flag uses truthiness in the source: setting it to `0` does not disable the judge. A provider key, a file named `live`, or a skipped suite is not proof that a model workflow completed.

### Dataset and capture demos

The compact script reads committed toy datasets and [local scoring code](lib/eval/mini.ts), independently of Jest discovery:

```bash
npm run -s demo:eval:compact
```

The package also defines `demo:eval`, `demo:capture-sse` and `demo:capture-sse:live`. They target Jest files under the excluded subtree, so they are not no-key first-run gates under the current configuration. See [package.json](package.json) and inspect the target file before choosing a provider-enabled run.

## 📊 Historical test summary (2025-09-17)

The following counts and metrics were recorded in the dated [master summary](TEST_MASTER_SUMMARY.md). They are archived reports, not newly reproduced results or current CI evidence.

- Suites: 18 passed, 1 skipped (19 total)
- Tests: 55 passed, 16 skipped (71 total)
- Skips: env‑gated live tests (require OPENAI_API_KEY)

Key signals
- Mini‑eval macro P/R/F1
  - Banking ≈ P 0.80, R 1.00, F1 0.867
  - AI Research ≈ P 0.767, R 0.933, F1 0.814
- Routes passed: ask, ask‑json, embeddings, agent, gather‑context, organization‑mode
- The archived summary reports successful live smokes where applicable; this handoff has not rerun them.

See also
- Dated write-up: [TEST_MASTER_SUMMARY.md](TEST_MASTER_SUMMARY.md)
- Archived preflight sample: [ai_research.preflight.sse.log](Interview/sse_samples/ai_research.preflight.sse.log)


## 🔐 Authentication

For simplicity, this public version uses a deterministic test-user system:

```typescript
// Default behavior
userId = "test-user"

// Override via header
headers: { 'x-test-user-id': 'custom-user-123' }

// Override via environment
PUBLIC_TEST_USER_ID=demo-user-456
```

**Production Note:** Replace `testUser.ts` with your actual auth middleware before deploying.

## 📊 Evaluation Framework

### Mini-Eval Example

The [compact script](scripts/demo_eval.ts) reads [banking](Interview/dataset/banking_toy_examples.json) and [AI research](Interview/dataset/ai_research_toy_examples.json) fixtures. Entries use the `Example` fields in [lib/eval/mini.ts](lib/eval/mini.ts): `id`, `prompt`, `bullets` and `expected_tags`.

Run `npm run -s demo:eval:compact` for the metrics from your checkout. The scorer uses tag/synonym string matching; those fixture scores do not measure general model capability.

### SSE Trace Analysis

Illustrative SSE event shapes (inspect the archived samples for their original context):

```jsonl
{"type":"tool_call","name":"search","args":{"query":"AGI safety"}}
{"type":"tool_result","data":{"hits":15,"top_result":"..."}}
{"type":"reasoning","text":"Found relevant papers, organizing..."}
{"type":"final","summary":"Created 3 notes with 15 citations"}
```

## 🚦 Environment Variables and integration limits

These names are read by the project source; values belong in the intended process environment:

| Variable | Source behavior |
|----------|-----------------|
| `OPENAI_API_KEY` | Controls Jest discovery; provider-using handlers also require it before import. Mocked route tests supply a dummy value internally. |
| `PUBLIC_TEST_USER_ID`, `NEXT_PUBLIC_HARDCODED_USER_ID` | Supply the deterministic test identity when no override header is used; they are not authentication. |
| `OPENAI_REAL_TESTS` | Additional opt-in for the saved-log judge in `live.smoke.eval.test.ts`; nonempty values are truthy. |
| `AGENT_PREFLIGHT` | Requests deterministic known-scenario handling; Jest worker presence also enables that path. |

Model choices are configured by the handler request schemas or source calls. There is no project-level `OPENAI_MODEL` environment read. SDK-specific environment behavior is not established by these notes.

The host application must supply runtime modules, service adapters and production authentication. The test mappings and deterministic user identity do not establish production readiness, ownership isolation or deployed endpoints.

## 🎓 Learning Resources

1. [Historical master summary](TEST_MASTER_SUMMARY.md) — dated test patterns and archived results.
2. [Interview guide](Interview/INTERVIEW_GUIDE.md) — walkthrough material.
3. [SSE samples](Interview/sse_samples/) — archived execution traces.
4. [Datasets](Interview/dataset/) — committed toy evaluation data.
5. [Change histories](CHANGELOG/README.md) — setup corrections, proof and limits.

## 🛠️ Development Workflow

1. Use the no-key mocked selection above for local handler work. `npm test -- --watch` invokes Jest watch mode with the same discovery rules.
2. Select an exact provider-backed suite only when that behavior is intended; review its flags and log prerequisites.
3. Integrate and verify handlers inside your actual host application before making deployment or authentication claims.

## 🤝 Contributing

We welcome contributions! Areas of interest:
- Additional tool integrations
- Evaluation metrics and datasets
- Provider adapters (Anthropic, Cohere, etc.)
- Performance optimizations

---

Start with the mocked harness, then verify the behavior and integration you actually intend to use.