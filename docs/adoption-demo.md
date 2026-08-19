# Five-Minute Adoption Demo

This document proposes a demo for maintainer and user validation. Its goal is to test whether Thunderbolt can communicate meaningful organizational value in five minutes, rather than only demonstrate that an LLM can summarize files.

## Demo Promise

> Give every approved AI model accurate, reusable company context without pasting the same policies, terminology, and product facts into every prompt.

The proposed scenario uses a fictional but realistic company context repository containing a product-plan matrix, security policy, support playbook, architecture summary, and glossary. Thunderbolt would connect to it through a read-only MCP server, retrieve relevant passages, and apply a reusable skill that requests sourced answers.

This creates a repeatable organizational capability for support, onboarding, engineering, sales, and operations.

## Five-Minute Journey

| Time      | User action                                                                  | Thunderbolt value                               |
| --------- | ---------------------------------------------------------------------------- | ----------------------------------------------- |
| 0:00–1:00 | Open a prepared deployment                                                   | Company-controlled AI entry point               |
| 1:00–2:00 | Inspect the Company Context connection and Company Answer skill              | Governed context and reusable behavior          |
| 2:00–3:00 | Ask whether an EU customer can export audit logs and what support should say | Selective, cited retrieval from company sources |
| 3:00–4:00 | Inspect the context and token receipt                                        | Visible sources, data flow, and efficiency      |
| 4:00–5:00 | Switch models and repeat the question                                        | Model portability without rebuilding context    |

The target answer cites the security policy, product plans, and support playbook; identifies conflicting or stale information; and recommends an owner. These are proposed success criteria for demonstrating provenance rather than fluent but unverifiable prose.

## Token-Savings Claim

Thunderbolt does not make tokens cheaper automatically. The expected saving comes from selective retrieval:

```text
Before: users repeatedly send complete documents
After:  Thunderbolt retrieves only relevant passages from a maintained source
```

Compare both paths with the same model:

1. Attach or paste all source documents.
2. Answer through the Company Context MCP server.
3. Compare input/output tokens, retrieved chunks, latency, estimated cost, citations, and required-fact coverage.

Do not promise a fixed saving before measuring representative content. Efficiency only matters if grounded answer quality remains acceptable.

## Existing Building Blocks

Thunderbolt already provides:

- Multi-provider and OpenAI-compatible model configuration
- MCP connections
- Reusable skills
- Attachments and text extraction
- Citation and document-result widgets
- Local conversation persistence
- An AI evaluation harness

The repository review did not identify a turnkey organizational context repository, production RAG service, or user-facing token receipt. Workspaces are described as future architecture. Any demo based on this proposal would need to distinguish proposed functionality from released functionality; maintainers may have additional roadmap context.

## Proposed Project

```text
examples/company-context/
├── README.md
├── context/
│   ├── company-glossary.md
│   ├── product-plans.md
│   ├── security-and-data-policy.md
│   ├── support-playbook.md
│   └── engineering-architecture.md
├── server/
│   ├── index.ts
│   ├── search.ts
│   └── search.test.ts
├── evaluations/
│   ├── questions.json
│   ├── required-facts.json
│   └── README.md
└── smoke.spec.ts
```

Use deterministic lexical or SQLite FTS retrieval initially. A vector database adds complexity without helping explain Thunderbolt's value.

Additional product work includes a seeded connection and skill, usage instrumentation, a context receipt, and baseline-versus-retrieval evaluation.

## One-Command Experience

The target interface is:

```sh
make demo
```

An implementation could use published images, start the sample context service, configure a demo identity, seed through supported interfaces, wait for health checks, and print one URL and one question. Provider keys must not enter fixtures or logs.

`make demo-local` can use Ollama or llama.cpp. Disclose model-download time separately rather than hiding it outside the five-minute claim.

## Path From Demo to Company Pilot

1. Choose one narrow, high-frequency question set.
2. Assign an owner and freshness metadata to every source.
3. Store context in a reviewed, version-controlled repository.
4. Expose authenticated, read-only search and fetch through MCP.
5. Require citations and explicit reporting of missing or conflicting context.
6. Evaluate accuracy, retrieval coverage, latency, and token use.
7. Pilot with a small group and use unanswered questions to improve sources.
8. Add write tools only after permissions, approvals, auditability, and prompt-injection risks are addressed.

## Follow-On Demos

1. Update a policy in Git and show the next answer cite the new version.
2. Compare two approved models against identical evidence.
3. Repeat with locally operated inference.
4. Demonstrate retrieval authorization boundaries.
5. Continue a conversation on another device using preview sync.
6. Inspect ciphertext in a disposable preview E2EE environment.

## Definition of Done

The demo achieves 5MTV when 90% of new evaluators can receive a correctly sourced company answer, inspect its context receipt, and repeat it with another model within five minutes, excluding a disclosed local-model download.

Track:

- Time to first sourced answer
- Completion rate and failure step
- Required-fact and citation coverage
- Baseline versus retrieved tokens and estimated cost
- Retrieval and total response latency
- Visible context version/freshness
- Attempts of a second question or model

A proposed validation path would run the complete flow in CI with deterministic retrieval and a mock model. Subject to maintainer approval, cost controls, and credential policy, a scheduled or release-gated test could use one real provider to catch proxy, tool-calling, usage, and streaming failures.
