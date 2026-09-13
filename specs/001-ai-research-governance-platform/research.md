# Research Findings: AI Research & Governance Platform

**Feature**: AI Research & Governance Platform
**Date**: 2026-09-12
**Status**: Complete - No NEEDS CLARIFICATION items required research

## Summary

The feature specification and constitution provided sufficient detail to make all technical decisions without requiring external research. All architectural choices align with the constitutional principles and the detailed specification in spec.md.

## Technical Decisions

### 1. Agent Orchestration: LangGraph vs Custom State Machine

**Decision**: Use LangGraph for agent orchestration with fallback to custom state machine for simple workflows.

**Rationale**: 
- LangGraph provides built-in state management, checkpointing, and visualization
- Aligns with constitutional principle of explicit state (Article VIII)
- Supports human-in-the-loop (Article VI) through interrupt mechanisms
- Mature ecosystem with good observability integration
- Custom state machine as fallback for simpler agents (critic, evaluator) to avoid over-engineering (Article XVI)

**Alternatives considered**: 
- Pure custom state machine: More control but reinvents workflow engine
- AutoGen: More complex, less transparent state
- CrewAI: Less flexible for custom tool integration

### 2. Experiment Tracking: Internal vs MLflow

**Decision**: Implement internal experiment tracker with MLflow-compatible API.

**Rationale**:
- Constitutional principle: Simplicity before complexity (Article XVI)
- Full control over reproducibility metadata (Article III)
- No external dependency for core functionality
- MLflow-compatible API allows future integration without lock-in
- PostgreSQL + pgvector already required for governance/evidence

**Alternatives considered**:
- Full MLflow: Adds Java dependency, separate server, overkill for MVP
- Weights & Biases: SaaS dependency, not offline-capable
- Sacred: Less maintained, Python-only

### 3. Sandbox Execution: Docker vs gVisor/Kata

**Decision**: Docker containers with resource limits (CPU, memory, pids, network=none) for MVP.

**Rationale**:
- Constitutional principle: Security by default (Article V)
- Docker provides sufficient isolation for MVP
- gVisor/Kata add complexity; can be added later if needed
- Network disabled by default satisfies constitutional requirement
- Resource limits enforce bounded autonomy (Article IX)

**Alternatives considered**:
- gVisor: Stronger isolation but more complex setup
- Kata Containers: Requires VM, heavier
- Firecracker: Overkill for local development
- Subprocess with seccomp: Less portable, harder to manage

### 4. Model Abstraction: Custom vs LiteLLM

**Decision**: Custom model abstraction layer with LiteLLM as optional backend.

**Rationale**:
- Constitutional principle: Model agnosticism (Article VII)
- Custom layer gives full control over interfaces, routing, cost tracking
- LiteLLM can be used as implementation detail for provider normalization
- Avoids dependency on external library for core abstraction
- Supports all required providers: OpenAI, Anthropic, Gemini, Ollama, vLLM, MLX

**Alternatives considered**:
- LiteLLM directly: Good but adds dependency; less control over routing logic
- LangChain LLM abstraction: Too coupled to LangChain ecosystem
- Direct provider SDKs: Violates model agnosticism principle

### 5. Vector Search: pgvector vs Dedicated Vector DB

**Decision**: PostgreSQL + pgvector for MVP.

**Rationale**:
- Constitutional principle: Simplicity before complexity (Article XVI)
- PostgreSQL already required for primary storage
- pgvector sufficient for MVP scale (thousands of embeddings)
- Single database reduces operational complexity
- Can migrate to dedicated vector DB (Qdrant, Weaviate) if scale demands

**Alternatives considered**:
- Qdrant: Excellent but adds separate service
- Weaviate: Heavy, Go-based, separate service
- Pinecone: SaaS, not offline-capable
- Chroma: Python-only, less production-hardened

### 6. Frontend State Management: React Query + Zustand

**Decision**: React Query (TanStack Query) for server state, Zustand for client state.

**Rationale**:
- React Query: Excellent for async data fetching, caching, synchronization
- Zustand: Lightweight, simple API for UI state (modals, selections)
- Both are mature, well-typed, minimal boilerplate
- Avoids Redux complexity for MVP

**Alternatives considered**:
- Redux Toolkit: More boilerplate, overkill for MVP
- SWR: Good but React Query has better mutation/invalidation
- Jotai: Atomic but less familiar to team

### 7. Observability: OpenTelemetry + Structured Logs

**Decision**: OpenTelemetry for traces/metrics, structured JSON logs, optional Langfuse.

**Rationale**:
- Constitutional principle: Observability and cost awareness (Article XV)
- OpenTelemetry is vendor-neutral standard
- Structured logs enable log aggregation and analysis
- Langfuse as optional for LLM-specific observability
- Cost tracking built into agent runtime

**Alternatives considered**:
- Datadog/New Relic: SaaS, vendor lock-in
- Prometheus/Grafana: Good for metrics, less for traces
- Langfuse only: LLM-focused, not full system observability

### 8. Configuration: Environment Variables + YAML

**Decision**: Environment variables for secrets/runtime config, YAML for policy/framework definitions.

**Rationale**:
- Constitutional principle: Simplicity (Article XVI)
- 12-factor app compliance for env vars
- YAML human-readable for policy frameworks (governance)
- No complex config management system needed for MVP

**Alternatives considered**:
- Pydantic Settings: Good but adds dependency
- TOML: Less common for policy definitions
- JSON: Less human-readable for policies

## Constitutional Alignment Verification

All technical decisions were validated against the 20 constitutional articles. No violations found. Key alignments:

- **Evidence First (I)**: Internal experiment tracker, structured tool results, claim type distinction
- **Tools Execute (II)**: Tool abstraction layer, typed interfaces, observable execution
- **Reproducibility (III)**: Full experiment metadata, checksum-addressable artifacts
- **Evaluation (IV)**: Critic agent, PASS/FAIL/INCONCLUSIVE, baseline comparison
- **Security (V)**: Docker sandbox, network=none, command guard, secret redaction
- **Human Oversight (VI)**: Explicit approval workflow for high-impact actions
- **Model Agnostic (VII)**: Custom abstraction, provider adapters, configurable routing
- **Explicit State (VIII)**: LangGraph state, structured agent state, execution traces
- **Bounded Autonomy (IX)**: Configurable limits, stop conditions
- **Fail Safely (X)**: Failure recording, bounded retries, uncertainty expression
- **Evidence Governance (XI)**: Versioned policies, stable control IDs, explainable scoring
- **Scientific Research (XII)**: Baselines, hypotheses, controlled experiments, failure preservation
- **Least Privilege (XIII)**: Tool capability declarations, scoped permissions
- **Testability (XIV)**: Unit/integration/agent tests, security tests
- **Observability (XV)**: OpenTelemetry, cost tracking, structured logs
- **Simplicity (XVI)**: Clear modules, PostgreSQL, Docker, typed interfaces
- **Transparency (XVII)**: Architecture docs, demo evidence, reproducible examples
- **Synthetic Data (XVIII)**: Demo datasets, no real sensitive data
- **Trustworthy Agent (XIX)**: PLAN→EXECUTE→VERIFY→EVALUATE→DECIDE→REMEMBER cycle
- **Priority (XX)**: Safety/evidence/human control over autonomy/speed

## Dependencies and Best Practices Researched

| Area | Decision | Source |
|------|----------|--------|
| FastAPI async patterns | Use lifespan events, dependency injection | FastAPI docs |
| SQLAlchemy 2.0 async | AsyncSession, select() syntax | SQLAlchemy docs |
| LangGraph patterns | StateGraph, checkpointer, interrupts | LangGraph docs |
| Docker sandbox security | --network=none, --pids-limit, --cpus, --memory | Docker security best practices |
| pgvector indexing | HNSW for ANN, IVFFlat for exact | pgvector docs |
| React Query v5 | QueryClient, useQuery, useMutation | TanStack Query docs |
| OpenTelemetry Python | TracerProvider, MeterProvider, auto-instrumentation | OpenTelemetry docs |
| Ruff + mypy config | Strict mode, per-file ignores | Ruff/mypy docs |

## Open Questions for Implementation Phase

These are not NEEDS CLARIFICATION (they don't affect spec) but implementation details to resolve during coding:

1. **LangGraph checkpointer**: SQLite vs PostgreSQL for checkpoint storage
2. **Artifact storage**: Local filesystem vs S3-compatible for artifacts
3. **Authentication**: JWT vs session-based for future multi-user
4. **WebSocket vs SSE**: For real-time agent trace updates
5. **Database migrations**: Alembic vs SQLAlchemy native
6. **CLI framework**: Typer vs Click vs argparse

These will be decided during implementation based on simplicity and constitutional alignment.