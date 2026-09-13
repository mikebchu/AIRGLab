# Implementation Plan: AI Research & Governance Platform

**Branch**: `001-ai-research-governance-platform` | **Date**: 2026-09-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-ai-research-governance-platform/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Build an agentic AI/ML engineering platform with four core capabilities: autonomous ML research (hypothesis-driven experimentation with evidence), AI governance assessment (risk analysis against versioned policy frameworks), red team security testing (safe adversarial testing in sandboxes), and evidence-backed reporting with full traceability. The platform demonstrates "Agents propose. Tools execute. Evidence decides." through deterministic tool execution, reproducible experiments, model-agnostic agent runtime, human-in-the-loop approvals, and security-by-default architecture.

## Technical Context

**Language/Version**: Python 3.12+ (backend), TypeScript 5+ (frontend)

**Primary Dependencies**: 
- Backend: FastAPI, Pydantic v2, SQLAlchemy 2.0, PostgreSQL, pgvector, LangGraph (or custom state machine), pandas, numpy, scikit-learn, XGBoost, PyTorch, OpenTelemetry, ruff, mypy, pytest
- Frontend: React 18, TypeScript, Vite, Recharts
- Infrastructure: Docker, Docker Compose

**Storage**: PostgreSQL (primary), pgvector (embeddings), file system (artifacts)

**Testing**: pytest, pytest-asyncio (backend), Vitest (frontend), security tests

**Target Platform**: Linux server (Docker), local development

**Project Type**: Web application (backend + frontend) with CLI

**Performance Goals**: 
- Agent run latency: <30s for simple tasks, <5min for complex research
- Experiment execution: sandboxed, resource-limited
- API response: <200ms p95 for non-streaming endpoints
- Concurrent agent runs: 10+ with resource isolation

**Constraints**: 
- Sandboxed execution for untrusted code
- Network access disabled by default for experiments
- Secrets never exposed to LLMs
- Human approval required for high-impact actions
- Synthetic data only for demos
- Model-agnostic: no provider lock-in

**Scale/Scope**: 
- Single-user local deployment for MVP
- 100+ experiments per project
- 1000+ agent events per run
- 10+ governance frameworks supported
- 15+ red team attack scenarios

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| Article I: Evidence Over Claims | ✅ PASS | Spec requires evidence for all claims, structured results, claim type distinction |
| Article II: Agents Decide; Tools Execute | ✅ PASS | Spec mandates tool abstraction, structured schemas, observable execution |
| Article III: Reproducibility First-Class | ✅ PASS | Spec requires full experiment metadata, immutable results |
| Article IV: Evaluation Before Acceptance | ✅ PASS | Spec requires success criteria, baselines, critic agent, PASS/FAIL/INCONCLUSIVE |
| Article V: Security by Default | ✅ PASS | Spec mandates sandboxing, scoped access, network disabled, secret redaction, command guard |
| Article VI: Human Oversight | ✅ PASS | Spec requires explicit approval for high-impact actions with recorded decisions |
| Article VII: Model Agnosticism | ✅ PASS | Spec requires provider abstraction, configurable model selection |
| Article VIII: Explicit State & Observability | ✅ PASS | Spec requires structured agent state, execution traces with full context |
| Article IX: Bounded Autonomy | ✅ PASS | Spec requires configurable limits, stop conditions |
| Article X: Fail Safely & Surface Uncertainty | ✅ PASS | Spec requires failure recording, bounded retries, uncertainty expression |
| Article XI: Evidence-Based Governance | ✅ PASS | Spec requires versioned policies, stable control IDs, explainable scoring |
| Article XII: Scientific Research | ✅ PASS | Spec requires baselines, hypotheses, controlled experiments, failure preservation |
| Article XIII: Least Privilege Tools | ✅ PASS | Spec requires capability declarations, scoped permissions, hierarchy |
| Article XIV: Testability | ✅ PASS | Spec requires automated tests for domain logic, policies, security, workflows |
| Article XV: Observability & Cost | ✅ PASS | Spec requires tracing, token/cost tracking at multiple levels |
| Article XVI: Simplicity Before Complexity | ✅ PASS | Spec prefers clear modules, typed interfaces, PostgreSQL, Docker over frameworks |
| Article XVII: Portfolio-Grade Transparency | ✅ PASS | Spec requires architecture docs, demo evidence, reproducible examples |
| Article XVIII: Synthetic Data Default | ✅ PASS | Spec mandates synthetic/public data for demos, no real sensitive data |
| Article XIX: Trustworthy Agent Definition | ✅ PASS | Spec implements PLAN→EXECUTE→VERIFY→EVALUATE→DECIDE→REMEMBER cycle |
| Article XX: Priority of Principles | ✅ PASS | Spec prioritizes safety, evidence, human control over autonomy/speed |

All 20 constitutional principles satisfied. No violations.

## Project Structure

### Documentation (this feature)

```text
specs/001-ai-research-governance-platform/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
# Web application (backend + frontend)
apps/
├── api/
│   ├── main.py
│   ├── routes/
│   │   ├── projects.py
│   │   ├── research.py
│   │   ├── experiments.py
│   │   ├── governance.py
│   │   ├── redteam.py
│   │   ├── runs.py
│   │   ├── evidence.py
│   │   ├── reports.py
│   │   └── models.py
│   ├── dependencies/
│   └── middleware/
├── web/
│   ├── src/
│   │   ├── components/
│   │   │   ├── dashboard/
│   │   │   ├── research/
│   │   │   ├── governance/
│   │   │   ├── redteam/
│   │   │   ├── trace/
│   │   │   └── common/
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Research.tsx
│   │   │   ├── Governance.tsx
│   │   │   ├── RedTeam.tsx
│   │   │   ├── Trace.tsx
│   │   │   ├── Models.tsx
│   │   │   └── Settings.tsx
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── utils/
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts

core/
├── config.py
├── logging.py
├── events.py
├── exceptions.py

agents/
├── base.py
├── director.py
├── research.py
├── governance.py
├── redteam.py
├── critic.py
├── evaluator.py

agent_runtime/
├── state.py
├── graph.py
├── router.py
├── memory.py
├── policies.py

tools/
├── base.py
├── filesystem.py
├── git.py
├── github.py
├── python.py
├── shell.py
├── dataset.py
├── profiling.py
├── training.py
├── evaluation.py
├── benchmarking.py
├── security.py
├── llm.py

research/
├── hypotheses.py
├── planner.py
├── experiments.py
├── metrics.py
├── comparison.py
├── research_memory.py
├── report.py

governance/
├── assessment.py
├── risk.py
├── controls.py
├── evidence.py
├── scoring.py
├── frameworks/
│   ├── nist_ai_rmf.yaml
│   ├── iso_42001.yaml
│   ├── enterprise_ai.yaml

redteam/
├── planner.py
├── attacks.py
├── scenarios.py
├── evaluator.py
├── report.py

evaluation/
├── evaluator.py
├── benchmarks.py
├── regression.py
├── scoring.py

memory/
├── models.py
├── repository.py
├── retrieval.py

storage/
├── database.py
├── artifacts.py

observability/
├── tracing.py
├── metrics.py
├── events.py

security/
├── sandbox.py
├── prompt_injection.py
├── secrets.py
├── command_guard.py
├── network_policy.py

cli/
├── main.py
├── commands/
│   ├── init.py
│   ├── research.py
│   ├── govern.py
│   ├── redteam.py
│   ├── run.py
│   ├── experiments.py
│   ├── report.py
│   └── test.py

tests/
├── unit/
├── integration/
├── agent/
├── research/
├── governance/
├── redteam/
├── security/

examples/
├── fraud_detection/
├── recommendation_system/
├── llm_chatbot/
├── broken_ml_project/

data/
├── demo/

artifacts/
├── .gitkeep
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
