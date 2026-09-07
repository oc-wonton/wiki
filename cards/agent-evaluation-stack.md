# Agent Evaluation Stack

## Core model

A reusable evaluation has four parts:

1. **Dataset** — versioned cases with input, setup/fixtures, expected invariants, tags, and provenance.
2. **Runner** — invokes the exact flow or external agent version under test in an isolated environment.
3. **Scorers** — deterministic checks first, then rubric/model grading only for semantic qualities.
4. **Experiment record** — pins dataset, prompt/skill, model, tools, code commit, outputs, traces, latency, cost, and scores.

Promotion compares a candidate against a blessed baseline. Block release on hard invariant failures and meaningful aggregate regressions. Production failures and human corrections become new dataset cases after review.

## Tool choice

### Inspect AI — default for tool-using agents and coding flows

Developed by the UK AI Security Institute and Meridian Labs. It models an eval as Dataset + Solver/Agent + Scorer, supports arbitrary external agents including Claude Code/Codex/Gemini, tool traces, multi-agent runs, and Docker/Kubernetes/other sandboxes. Use for OMP coding tasks, tool trajectories, repository fixtures, and security boundaries.

`meridianlabs-ai/inspect-skills` provides four coding-agent skills for package selection, reading/analyzing logs, and monitoring live evals. It assists operation and analysis; Inspect AI remains the actual eval runner.

### Promptfoo — default for prompt/API/content flows

Local open-source CLI with declarative YAML/JSON test cases, assertions, model/prompt matrices, caching, CI and red teaming. Use for AI Daily selection/deduplication, classifiers, summaries, structured output, and API-backed workflows.

### Braintrust or LangSmith — when hosted tracing and collaboration are needed

Useful for production traces, online scoring, browser datasets and team dashboards. They add operational convenience but introduce hosted state and vendor dependency; not required for an initial local eval system.

## Dataset shape

Each case should include:

```yaml
id: stable-case-id
input: task payload or user request
fixture: repository snapshot or frozen source bundle
expected:
  must_pass: [deterministic assertions]
  must_not: [forbidden behavior]
  rubric: semantic quality criteria
metadata:
  category: duplicate|security|bugfix|no-news
  risk: low|high
  source: production failure|manual|synthetic
```

Do not require exact prose when many answers are valid. Prefer executable outcomes: tests, schemas, file diffs, URLs, tool permissions, source timestamps, and absence of forbidden changes.

## Evaluation lifecycle

1. Start with 5–10 manually curated cases per critical capability.
2. Freeze input fixtures; never let live web data or a moving repository make the baseline irreproducible.
3. Run each candidate more than once for nondeterministic tasks and report pass-rate distribution.
4. Keep a stable holdout set separate from the cases used while tuning prompts.
5. Record model, thinking level, prompt/skill hash, tool policy, code commit, cost, latency, outputs, and traces.
6. Require zero failures on hard safety/correctness checks; compare quality, cost and latency against the baseline.
7. Review production failures and add representative, privacy-safe cases to the dataset.
8. Version dataset changes separately from system changes so regressions remain attributable.

## Recommended local split

- AI Daily: Promptfoo with frozen candidate feeds and deterministic dedupe/source/time-window assertions.
- OMP coding: Inspect AI with small bug repositories or patches, containerized execution, tests/security checks, diff-size checks, and trajectory inspection.
- AI-Native SDLC: deterministic schema/revision-chain tests plus a small semantic regression suite for intent/spec/plan quality.
