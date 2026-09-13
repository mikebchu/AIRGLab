# Tasks: AI Research & Governance Platform

**Input**: Design documents from `/specs/001-ai-research-governance-platform/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are OPTIONAL - only include them if explicitly requested in the feature specification.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

Based on plan.md project structure:
- Backend: `apps/api/`
- Frontend: `apps/web/`
- Core: `core/`
- Agents: `agents/`
- Agent Runtime: `agent_runtime/`
- Tools: `tools/`
- Research: `research/`
- Governance: `governance/`
- Red Team: `redteam/`
- Evaluation: `evaluation/`
- Memory: `memory/`
- Storage: `storage/`
- Observability: `observability/`
- Security: `security/`
- CLI: `cli/`
- Tests: `tests/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project directory structure per implementation plan (apps/api, apps/web, core, agents, agent_runtime, tools, research, governance, redteam, evaluation, memory, storage, observability, security, cli, tests, examples, data, artifacts)
- [ ] T002 Initialize Python project with pyproject.toml (FastAPI, Pydantic v2, SQLAlchemy 2.0, PostgreSQL, pgvector, LangGraph, pandas, numpy, scikit-learn, XGBoost, PyTorch, OpenTelemetry, ruff, mypy, pytest, pytest-asyncio)
- [ ] T003 [P] Initialize Node.js project in apps/web with package.json (React 18, TypeScript, Vite, Recharts)
- [ ] T004 [P] Configure Docker Compose with services: api, web, db (PostgreSQL+pgvector), redis
- [ ] T005 [P] Configure linting and formatting: ruff, mypy, prettier, eslint
- [ ] T006 [P] Create .env.example with all required environment variables (LLM_PROVIDER, LLM_MODEL, LLM_API_KEY, LLM_BASE_URL, DATABASE_URL, REDIS_URL, API_KEY, APPROVAL_MODE, SANDBOX_ENABLED, NETWORK_ACCESS, MAX_EXPERIMENTS, MAX_COST_USD)
- [ ] T007 [P] Create .gitignore for Python, Node, Docker, IDE files
- [ ] T008 [P] Create README.md with quickstart instructions

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Database & Storage
- [ ] T009 Setup database connection and session management in storage/database.py (SQLAlchemy 2.0 async engine, session factory, health check)
- [ ] T010 Create database migrations framework (Alembic) with initial migration for all 17 entities from data-model.md
- [ ] T011 [P] Implement Project model in storage/models/project.py with fields: id (UUID, PK), name (String, NOT NULL, max 255), description (Text, NULLABLE), repository_url (String, NULLABLE, max 500), created_at, updated_at
- [ ] T012 [P] Implement ResearchQuestion model in storage/models/research_question.py with fields: id, project_id (FK), question (Text, NOT NULL), objective (Text, NOT NULL), constraints (JSONB, default '{}'), success_criteria (JSONB, default '{}'), status (Enum: pending/running/completed/failed/cancelled), created_at, updated_at
- [ ] T013 [P] Implement Hypothesis model in storage/models/hypothesis.py with fields: id, research_question_id (FK), statement (Text, NOT NULL), rationale (Text, NULLABLE), expected_outcome (JSONB, default '{}'), confidence (Float, default 0.5), status (Enum: proposed/testing/accepted/rejected/inconclusive), created_at, updated_at
- [ ] T014 [P] Implement Experiment model in storage/models/experiment.py with fields: id, hypothesis_id (FK), name (String, NOT NULL, max 255), description (Text, NULLABLE), dataset_version (String, NOT NULL, max 255), code_commit (String, NOT NULL, max 64), model_name (String, NOT NULL, max 255), model_version (String, NULLABLE, max 255), parameters (JSONB, default '{}'), seed (Integer, NULLABLE), status (Enum: pending/running/completed/failed/cancelled), started_at, completed_at, duration_seconds, cost_usd, error_message, created_at, updated_at
- [ ] T015 [P] Implement Metric model in storage/models/metric.py with fields: id, experiment_id (FK), name (String, NOT NULL, max 100), value (Float, NOT NULL), unit (String, NULLABLE, max 50), direction (Enum: higher_better/lower_better/target), baseline_value (Float, NULLABLE), delta (Float, NULLABLE), created_at
- [ ] T016 [P] Implement ExperimentArtifact model in storage/models/experiment_artifact.py with fields: id, experiment_id (FK), artifact_type (Enum: model/plot/log/dataset/report/other), path (String, NOT NULL, max 1000), checksum (String, NOT NULL, max 64), size_bytes (Integer, NOT NULL), metadata (JSONB, default '{}'), created_at
- [ ] T017 [P] Implement GovernanceAssessment model in storage/models/governance_assessment.py with fields: id, project_id (FK), system_name (String, NOT NULL, max 255), system_description (Text, NULLABLE), architecture (JSONB, default '{}'), data_inventory (JSONB, default '{}'), models (JSONB, default '[]'), vendors (JSONB, default '[]'), deployment_regions (JSONB, default '[]'), user_populations (JSONB, default '[]'), intended_use (Text, NULLABLE), prohibited_use (Text, NULLABLE), existing_controls (JSONB, default '[]'), framework (String, NOT NULL, max 100), framework_version (String, NOT NULL, max 50), risk_level (Enum: critical/high/medium/low/informational), risk_score (Float, NOT NULL), risk_breakdown (JSONB, default '{}'), status (Enum: pending/running/completed/failed), created_at, updated_at
- [ ] T018 [P] Implement GovernanceControl model in storage/models/governance_control.py with fields: id, framework (String, NOT NULL, max 100), framework_version (String, NOT NULL, max 50), control_id (String, NOT NULL, max 100), title (String, NOT NULL, max 255), description (Text, NOT NULL), severity (Enum: critical/high/medium/low/informational), category (String, NOT NULL, max 100), requirements (JSONB, default '[]'), evidence_types (JSONB, default '[]'), pass_condition (Text, NOT NULL), created_at, updated_at. Unique constraint on (framework, framework_version, control_id)
- [ ] T019 [P] Implement Finding model in storage/models/finding.py with fields: id, assessment_id (FK), control_id (FK, NULLABLE), severity (Enum: critical/high/medium/low/informational), category (String, NOT NULL, max 100), title (String, NOT NULL, max 255), description (Text, NOT NULL), recommendation (Text, NULLABLE), status (Enum: open/in_progress/resolved/accepted_risk/false_positive), architecture_component (String, NULLABLE, max 255), created_at, updated_at
- [ ] T020 [P] Implement Evidence model in storage/models/evidence.py with fields: id, source_type (Enum: tool_output/document/code/dataset/model/manual/external), source_reference (String, NOT NULL, max 500), content_hash (String, NOT NULL, max 64), content_preview (Text, NULLABLE), collected_at, reliability (Enum: high/medium/low/unknown), related_control_id (FK, NULLABLE), related_finding_id (FK, NULLABLE), related_experiment_id (FK, NULLABLE), related_redteam_test_id (FK, NULLABLE), metadata (JSONB, default '{}'), created_at
- [ ] T021 [P] Implement RedTeamRun model in storage/models/redteam_run.py with fields: id, project_id (FK), target_system (String, NOT NULL, max 255), target_description (Text, NULLABLE), target_type (Enum: llm_app/agent/api/prompt/tool/data_flow), status (Enum: pending/running/completed/failed), started_at, completed_at, created_at, updated_at
- [ ] T022 [P] Implement RedTeamTest model in storage/models/redteam_test.py with fields: id, redteam_run_id (FK), scenario (String, NOT NULL, max 255), attack_type (Enum: prompt_injection/tool_abuse/secret_exposure/command_injection/path_traversal/data_poisoning/hallucination/unsafe_recommendation/context_manipulation/goal_hijacking/system_prompt_extraction/sensitive_disclosure/unauthorized_tool/excessive_permissions/indirect_injection), input (Text, NOT NULL), expected_behavior (Text, NOT NULL), actual_behavior (Text, NULLABLE), result (Enum: pass/fail/inconclusive/error), severity (Enum: critical/high/medium/low/informational), evidence_ids (UUID[], default '[]'), created_at, updated_at
- [ ] T023 [P] Implement AgentRun model in storage/models/agent_run.py with fields: id, agent_type (Enum: research_director/dataset_analyst/research_planner/experiment_planner/experiment_executor/evaluator/critic/governance_analyst/redteam_planner/redteam_executor), objective (Text, NOT NULL), status (Enum: pending/running/completed/failed/cancelled/awaiting_approval), parent_run_id (FK, NULLABLE), project_id (FK), research_question_id (FK, NULLABLE), hypothesis_id (FK, NULLABLE), experiment_id (FK, NULLABLE), governance_assessment_id (FK, NULLABLE), redteam_run_id (FK, NULLABLE), started_at, completed_at, total_tokens, estimated_cost_usd, error_message, created_at, updated_at
- [ ] T024 [P] Implement AgentEvent model in storage/models/agent_event.py with fields: id, agent_run_id (FK), timestamp, event_type (Enum: planning/tool_call/tool_result/decision/error/approval_request/approval_response/evaluation/critique/memory_store/memory_retrieve), message (Text, NULLABLE), tool_name (String, NULLABLE, max 100), tool_input (JSONB, NULLABLE), tool_output (JSONB, NULLABLE), status (Enum: success/failure/partial/pending), duration_ms, evidence_id (FK, NULLABLE), metadata (JSONB, default '{}')
- [ ] T025 [P] Implement Decision model in storage/models/decision.py with fields: id, agent_run_id (FK), decision_type (Enum: continue/modify_hypothesis/new_hypothesis/stop/approve/reject/request_human), decision (Text, NOT NULL), rationale (Text, NOT NULL), evidence_ids (UUID[], default '[]'), confidence (Float, default 0.5), requires_human_approval (Boolean, default false), approval_status (Enum: pending/approved/rejected, NULLABLE), approved_by (String, NULLABLE, max 255), approved_at, created_at
- [ ] T026 [P] Implement ResearchMemory model in storage/models/research_memory.py with fields: id, project_id (FK), statement (Text, NOT NULL), confidence (Float, default 0.5), evidence_ids (UUID[], default '[]'), tags (String[], default '[]'), source_type (Enum: hypothesis/experiment/failure/conclusion/lesson), source_id (UUID, NULLABLE), created_at, updated_at
- [ ] T027 Create database indexes for performance: idx_research_question_project, idx_hypothesis_research_question, idx_experiment_hypothesis, idx_metric_experiment, idx_artifact_experiment, idx_assessment_project, idx_finding_assessment, idx_evidence_finding, idx_evidence_experiment, idx_redteam_run_project, idx_redteam_test_run, idx_agent_run_project, idx_agent_event_run, idx_decision_run, idx_memory_project
- [ ] T028 Implement artifact storage in storage/artifacts.py (local filesystem with SHA256 checksums, metadata JSON)

### Core Infrastructure
- [ ] T029 Implement configuration management in core/config.py (Pydantic Settings with env vars, YAML config file support, validation)
- [ ] T030 Implement structured logging in core/logging.py (JSON format, correlation IDs, log levels)
- [ ] T031 Implement event system in core/events.py (event bus for agent events, WebSocket broadcasting)
- [ ] T032 Implement custom exceptions in core/exceptions.py (ValidationError, NotFoundError, AuthorizationError, BudgetExceededError, SandboxError, ToolError, ApprovalRequiredError, ApprovalRejectedError)
- [ ] T033 [P] Create database seed script for governance frameworks (NIST AI RMF, ISO 42001, Enterprise AI controls from governance/frameworks/*.yaml)

### Security Infrastructure
- [ ] T034 Implement sandbox execution in security/sandbox.py (Docker-based with resource limits: CPU, memory, timeout, network=none, read-only filesystem except /workspace)
- [ ] T035 Implement command guard in security/command_guard.py (blocklist: rm -rf, sudo, chmod 777, curl/wget to external, credential access patterns)
- [ ] T036 Implement secret redaction in security/secrets.py (redact API keys, passwords, tokens from logs and model-visible output)
- [ ] T037 Implement prompt injection detection in security/prompt_injection.py (heuristic + classifier for untrusted input)
- [ ] T038 Implement network policy in security/network_policy.py (deny by default, allowlist for approved domains)

### Model Abstraction
- [ ] T039 Implement model abstraction layer in tools/llm.py (BaseModelClient interface, providers: OpenAI, Anthropic, Gemini, Ollama, vLLM, MLX with OpenAI-compatible endpoints)
- [ ] T040 Implement model routing in agent_runtime/router.py (task-type based routing: research_planning→high_reasoning, code_generation→code_model, evaluation→cost_effective, local_only→privacy_sensitive)

### Observability
- [ ] T041 Implement OpenTelemetry tracing in observability/tracing.py (spans for agent runs, tool calls, experiments, HTTP requests)
- [ ] T042 Implement metrics collection in observability/metrics.py (Prometheus metrics: agent_runs_total, experiment_duration, token_usage, cost_usd, approval_requests)
- [ ] T043 Implement structured event logging in observability/events.py (AgentEvent persistence, WebSocket streaming)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Autonomous ML Research (Priority: P1) 🎯 MVP

**Goal**: Researcher submits ML research question with dataset and constraints; system autonomously profiles dataset, generates hypotheses, plans/executes experiments, evaluates results, challenges conclusions via critic, produces evidence-backed report.

**Independent Test**: Submit research question with synthetic fraud dataset → verify complete research report with experiments, evidence, conclusions.

### Tests for User Story 1 (OPTIONAL)

- [ ] T044 [P] [US1] Contract test for POST /projects/{id}/research in tests/contract/test_research.py
- [ ] T045 [P] [US1] Contract test for GET /projects/{id}/research/{id}/report in tests/contract/test_research.py
- [ ] T046 [P] [US1] Contract test for WS /projects/{id}/runs/{id}/events/stream in tests/contract/test_research.py
- [ ] T047 [P] [US1] Integration test for full research workflow in tests/integration/test_research_workflow.py (synthetic fraud dataset)

### Models & Database (US1)
- [ ] T048 [P] [US1] Create ResearchQuestion repository in storage/repository/research_question.py (CRUD, status transitions)
- [ ] T049 [P] [US1] Create Hypothesis repository in storage/repository/hypothesis.py (CRUD, status transitions)
- [ ] T050 [P] [US1] Create Experiment repository in storage/repository/experiment.py (CRUD, metrics, artifacts)
- [ ] T051 [P] [US1] Create Metric repository in storage/repository/metric.py (CRUD, comparison queries)
- [ ] T052 [P] [US1] Create ExperimentArtifact repository in storage/repository/experiment_artifact.py (CRUD, download)

### Tools (US1)
- [ ] T053 [P] [US1] Implement dataset profiling tool in tools/profiling.py (rows, columns, missingness, class balance, leakage detection - deterministic, no LLM)
- [ ] T054 [P] [US1] Implement Python execution tool in tools/python.py (sandboxed, resource-limited, returns stdout/stderr/artifacts)
- [ ] T055 [P] [US1] Implement shell execution tool in tools/shell.py (sandboxed, command guard, scoped filesystem)
- [ ] T056 [P] [US1] Implement training tool in tools/training.py (scikit-learn/XGBoost/PyTorch model training with MLflow-style logging)
- [ ] T057 [P] [US1] Implement evaluation tool in tools/evaluation.py (metrics computation, baseline comparison, statistical significance)
- [ ] T058 [P] [US1] Implement benchmarking tool in tools/benchmarking.py (cross-validation, hyperparameter search, model comparison)

### Research Agents (US1)
- [ ] T059 [P] [US1] Implement DatasetAnalyst agent in agents/research/dataset_analyst.py (profiles dataset, identifies features, suggests preprocessing)
- [ ] T060 [P] [US1] Implement ResearchPlanner agent in agents/research/research_planner.py (generates hypotheses with rationale, expected outcomes, confidence)
- [ ] T061 [P] [US1] Implement ExperimentPlanner agent in agents/research/experiment_planner.py (designs experiments: model selection, parameters, seeds, success criteria)
- [ ] T062 [P] [US1] Implement ExperimentExecutor agent in agents/research/experiment_executor.py (executes experiments via tools, records metadata per FR-005)
- [ ] T063 [P] [US1] Implement Evaluator agent in agents/research/evaluator.py (evaluates results against success criteria, PASS/FAIL/INCONCLUSIVE)
- [ ] T064 [P] [US1] Implement Critic agent in agents/critic.py (challenges conclusions, identifies insufficient evidence, suggests alternatives per FR-007)
- [ ] T065 [P] [US1] Implement ResearchDirector agent in agents/director.py (orchestrates research workflow: profile→baseline→hypotheses→experiments→evaluate→critic→report)
- [ ] T066 [P] [US1] Implement ResearchMemory in memory/research_memory.py (store/retrieve hypotheses, experiments, results, failures, lessons with relevance scoring)

### Research Services (US1)
- [ ] T067 [US1] Implement ResearchService in research/service.py (coordinates agents, manages research lifecycle, budget tracking per FR-023)
- [ ] T068 [US1] Implement ExperimentService in research/experiments.py (experiment CRUD, execution, comparison per contracts/api.md)
- [ ] T069 [US1] Implement ReportGenerator in research/report.py (generates final report with executive summary, methodology, experiments, results, evidence, failures, limitations, reproducibility, claim type distinction per FR-019)

### API Routes (US1)
- [ ] T070 [P] [US1] Implement POST /projects/{id}/research in apps/api/routes/research.py (create research question)
- [ ] T071 [P] [US1] Implement GET /projects/{id}/research in apps/api/routes/research.py (list with filters)
- [ ] T072 [P] [US1] Implement GET /projects/{id}/research/{id} in apps/api/routes/research.py (get by ID)
- [ ] T073 [P] [US1] Implement POST /projects/{id}/research/{id}/run in apps/api/routes/research.py (start research run)
- [ ] T074 [P] [US1] Implement GET /projects/{id}/research/{id}/status in apps/api/routes/research.py (get status)
- [ ] T075 [P] [US1] Implement GET /projects/{id}/research/{id}/report in apps/api/routes/research.py (get report)
- [ ] T076 [P] [US1] Implement GET /projects/{id}/experiments in apps/api/routes/experiments.py (list with filters)
- [ ] T077 [P] [US1] Implement GET /projects/{id}/experiments/{id} in apps/api/routes/experiments.py (get experiment with metrics/artifacts)
- [ ] T078 [P] [US1] Implement GET /projects/{id}/experiments/{id}/artifacts in apps/api/routes/experiments.py (list artifacts)
- [ ] T079 [P] [US1] Implement GET /projects/{id}/experiments/{id}/artifacts/{aid}/download in apps/api/routes/experiments.py (download artifact)
- [ ] T080 [P] [US1] Implement GET /projects/{id}/hypotheses in apps/api/routes/research.py (list hypotheses)

### CLI Commands (US1)
- [ ] T081 [P] [US1] Implement lab init in cli/commands/init.py (project initialization)
- [ ] T082 [P] [US1] Implement lab research in cli/commands/research.py (run research with all options)
- [ ] T083 [P] [US1] Implement lab research status in cli/commands/research.py (status with --watch)
- [ ] T084 [P] [US1] Implement lab research list in cli/commands/research.py (list with filters)
- [ ] T085 [P] [US1] Implement lab research report in cli/commands/research.py (report with format options)
- [ ] T086 [P] [US1] Implement lab experiments list in cli/commands/experiments.py (list with filters)
- [ ] T087 [P] [US1] Implement lab experiments show in cli/commands/experiments.py (show details)
- [ ] T088 [P] [US1] Implement lab experiments compare in cli/commands/experiments.py (compare two experiments)

### Frontend (US1)
- [ ] T089 [P] [US1] Create Research page in apps/web/src/pages/Research.tsx (question input, constraints, dataset upload, run button)
- [ ] T090 [P] [US1] Create ResearchStatus component in apps/web/src/components/research/ResearchStatus.tsx (live status, progress, current phase)
- [ ] T091 [P] [US1] Create ExperimentList component in apps/web/src/components/research/ExperimentList.tsx (table with metrics, status, actions)
- [ ] T092 [P] [US1] Create ExperimentDetail component in apps/web/src/components/research/ExperimentDetail.tsx (metrics, artifacts, comparison)
- [ ] T093 [P] [US1] Create ResearchReport component in apps/web/src/components/research/ResearchReport.tsx (render report with claim type badges)
- [ ] T094 [P] [US1] Create TraceViewer component in apps/web/src/components/trace/TraceViewer.tsx (chronological events, tool I/O, evidence links per FR-018)
- [ ] T095 [US1] Implement WebSocket hook in apps/web/src/hooks/useAgentTrace.ts (connect to WS /projects/{id}/runs/{id}/events/stream)
- [ ] T096 [US1] Create API service in apps/web/src/services/api.ts (typed API client for research endpoints)

**Checkpoint**: At this point, User Story 1 should be fully functional and testably independently

---

## Phase 4: User Story 2 - AI Governance Assessment (Priority: P1)

**Goal**: User submits AI system description; system decomposes system, maps data flows, identifies AI components, classifies risks, maps controls from frameworks, collects evidence, performs gap analysis, calculates risk scores, generates report with remediation plan.

**Independent Test**: Submit synthetic AI system description → verify governance report with risk scores, findings linked to controls/evidence, remediation plan.

### Tests for User Story 2 (OPTIONAL)

- [ ] T097 [P] [US2] Contract test for POST /projects/{id}/governance in tests/contract/test_governance.py
- [ ] T098 [P] [US2] Contract test for GET /projects/{id}/governance/{id} in tests/contract/test_governance.py
- [ ] T099 [P] [US2] Contract test for GET /governance/frameworks in tests/contract/test_governance.py
- [ ] T100 [P] [US2] Integration test for governance workflow in tests/integration/test_governance_workflow.py (cosmetics recommendation demo)

### Models & Database (US2)
- [ ] T101 [P] [US2] Create GovernanceAssessment repository in storage/repository/governance_assessment.py (CRUD, status transitions)
- [ ] T102 [P] [US2] Create GovernanceControl repository in storage/repository/governance_control.py (CRUD, framework/version queries)
- [ ] T103 [P] [US2] Create Finding repository in storage/repository/finding.py (CRUD, severity filtering, control linking)
- [ ] T104 [P] [US2] Create Evidence repository in storage/repository/evidence.py (CRUD, multi-entity linking)

### Governance Frameworks (US2)
- [ ] T105 [P] [US2] Create NIST AI RMF framework YAML in governance/frameworks/nist_ai_rmf.yaml (42 controls with stable IDs, severity, requirements, evidence types, pass conditions)
- [ ] T106 [P] [US2] Create ISO 42001 framework YAML in governance/frameworks/iso_42001.yaml (38 controls)
- [ ] T107 [P] [US2] Create Enterprise AI framework YAML in governance/frameworks/enterprise_ai.yaml (25 controls)
- [ ] T108 [P] [US2] Implement framework loader in governance/frameworks/loader.py (load YAML, validate schema, version management)

### Governance Agents (US2)
- [ ] T109 [P] [US2] Implement GovernanceAnalyst agent in agents/governance/governance_analyst.py (decomposes system, maps data flows, identifies AI components, classifies risks across 11 categories per FR-011)
- [ ] T110 [P] [US2] Implement ControlMapper agent in governance/controls.py (maps framework controls to system components, identifies applicable controls)
- [ ] T111 [P] [US2] Implement EvidenceLinker agent in governance/evidence.py (collects evidence for controls, assesses reliability, links to findings)
- [ ] T112 [P] [US2] Implement RiskScorer agent in governance/scoring.py (calculates explainable risk scores with per-category breakdown per FR-013)
- [ ] T113 [P] [US2] Implement RemediationPlanner agent in governance/assessment.py (generates prioritized remediation plan with effort estimates)

### Governance Services (US2)
- [ ] T114 [US2] Implement GovernanceService in governance/service.py (coordinates agents, manages assessment lifecycle)
- [ ] T115 [US2] Implement GovernanceReportGenerator in governance/report.py (generates report with overall risk score, findings linked to controls/evidence, severity ratings, remediation plan per FR-014)

### API Routes (US2)
- [ ] T116 [P] [US2] Implement POST /projects/{id}/governance in apps/api/routes/governance.py (create assessment)
- [ ] T117 [P] [US2] Implement GET /projects/{id}/governance in apps/api/routes/governance.py (list with filters)
- [ ] T118 [P] [US2] Implement GET /projects/{id}/governance/{id} in apps/api/routes/governance.py (get assessment with findings/evidence)
- [ ] T119 [P] [US2] Implement GET /governance/frameworks in apps/api/routes/governance.py (list frameworks)
- [ ] T120 [P] [US2] Implement GET /governance/frameworks/{id}/controls in apps/api/routes/governance.py (list controls with version)

### CLI Commands (US2)
- [ ] T121 [P] [US2] Implement lab govern in cli/commands/govern.py (run assessment with framework selection)
- [ ] T122 [P] [US2] Implement lab govern show in cli/commands/govern.py (show assessment)
- [ ] T123 [P] [US2] Implement lab govern list in cli/commands/govern.py (list with filters)
- [ ] T124 [P] [US2] Implement lab govern report in cli/commands/govern.py (report with format options)

### Frontend (US2)
- [ ] T125 [P] [US2] Create Governance page in apps/web/src/pages/Governance.tsx (system description input, framework selection, run button)
- [ ] T126 [P] [US2] Create SystemDecomposition component in apps/web/src/components/governance/SystemDecomposition.tsx (architecture, data flows, AI components visualization)
- [ ] T127 [P] [US2] Create RiskBreakdown component in apps/web/src/components/governance/RiskBreakdown.tsx (radar chart for 11 risk categories)
- [ ] T128 [P] [US2] Create FindingsTable component in apps/web/src/components/governance/FindingsTable.tsx (findings with control links, evidence, severity)
- [ ] T129 [P] [US2] Create GovernanceReport component in apps/web/src/components/governance/GovernanceReport.tsx (full report with remediation plan)
- [ ] T130 [US2] Add governance API methods to apps/web/src/services/api.ts

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Red Team Security Testing (Priority: P1)

**Goal**: User submits AI system target; system generates attack scenarios, executes tests in safe sandbox, evaluates responses, identifies vulnerabilities with evidence, produces security report with mitigations.

**Independent Test**: Submit vulnerable demo AI assistant → verify identification of ≥3 vulnerabilities (prompt injection, tool auth, secret protection) without real credential access.

### Tests for User Story 3 (OPTIONAL)

- [ ] T131 [P] [US3] Contract test for POST /projects/{id}/redteam in tests/contract/test_redteam.py
- [ ] T132 [P] [US3] Contract test for GET /projects/{id}/redteam/{id} in tests/contract/test_redteam.py
- [ ] T133 [P] [US3] Contract test for GET /redteam/scenarios in tests/contract/test_redteam.py
- [ ] T134 [P] [US3] Integration test for red team workflow in tests/integration/test_redteam_workflow.py (vulnerable chatbot demo)

### Models & Database (US3)
- [ ] T135 [P] [US3] Create RedTeamRun repository in storage/repository/redteam_run.py (CRUD, status transitions)
- [ ] T136 [P] [US3] Create RedTeamTest repository in storage/repository/redteam_test.py (CRUD, scenario filtering, severity queries)

### Red Team Scenarios (US3)
- [ ] T137 [P] [US3] Implement attack scenarios in redteam/scenarios.py (15 scenarios per FR-016: prompt_injection, system_prompt_extraction, sensitive_disclosure, tool_abuse, unauthorized_tool, command_injection, path_traversal, secret_exposure, excessive_permissions, indirect_injection, data_poisoning, hallucination, unsafe_recommendation, context_manipulation, goal_hijacking)
- [ ] T138 [P] [US3] Create scenario definitions YAML in redteam/scenarios.yaml (input templates, expected behaviors, severity mappings)

### Red Team Agents (US3)
- [ ] T139 [P] [US3] Implement RedTeamPlanner agent in agents/redteam/redteam_planner.py (selects scenarios based on target type, generates attack inputs)
- [ ] T140 [P] [US3] Implement RedTeamExecutor agent in agents/redteam/redteam_executor.py (executes attacks in sandbox, captures responses, evaluates results)
- [ ] T141 [P] [US3] Implement RedTeamEvaluator agent in redteam/evaluator.py (evaluates responses, classifies result pass/fail/inconclusive, assigns severity per FR-017)

### Red Team Services (US3)
- [ ] T142 [US3] Implement RedTeamService in redteam/service.py (coordinates agents, manages run lifecycle, enforces sandbox isolation)
- [ ] T143 [US3] Implement RedTeamReportGenerator in redteam/report.py (generates report with findings, evidence, mitigations)

### API Routes (US3)
- [ ] T144 [P] [US3] Implement POST /projects/{id}/redteam in apps/api/routes/redteam.py (create run)
- [ ] T145 [P] [US3] Implement GET /projects/{id}/redteam in apps/api/routes/redteam.py (list with filters)
- [ ] T146 [P] [US3] Implement GET /projects/{id}/redteam/{id} in apps/api/routes/redteam.py (get run with tests/findings)
- [ ] T147 [P] [US3] Implement GET /redteam/scenarios in apps/api/routes/redteam.py (list attack scenarios)

### CLI Commands (US3)
- [ ] T148 [P] [US3] Implement lab redteam in cli/commands/redteam.py (run assessment with scenario selection)
- [ ] T149 [P] [US3] Implement lab redteam show in cli/commands/redteam.py (show results)
- [ ] T150 [P] [US3] Implement lab redteam list in cli/commands/redteam.py (list with filters)
- [ ] T151 [P] [US3] Implement lab redteam report in cli/commands/redteam.py (report with format options)

### Frontend (US3)
- [ ] T152 [P] [US3] Create RedTeam page in apps/web/src/pages/RedTeam.tsx (target input, scenario selection, run button)
- [ ] T153 [P] [US3] Create AttackScenarioList component in apps/web/src/components/redteam/AttackScenarioList.tsx (scenarios with descriptions, severity)
- [ ] T154 [P] [US3] Create TestResults component in apps/web/src/components/redteam/TestResults.tsx (test table with pass/fail, severity, evidence)
- [ ] T155 [P] [US3] Create RedTeamReport component in apps/web/src/components/redteam/RedTeamReport.tsx (vulnerability summary, mitigations)
- [ ] T156 [US3] Add redteam API methods to apps/web/src/services/api.ts

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently

---

## Phase 6: User Story 4 - Evidence-Backed Reporting & Traceability (Priority: P1)

**Goal**: Every agent action produces structured evidence and traces. Users can inspect traces, view experiment evidence, verify reproducibility metadata, distinguish claim types.

**Independent Test**: Run any workflow → verify trace UI shows chronological events with tool I/O, evidence IDs, decisions, report distinguishes claim types.

### Tests for User Story 4 (OPTIONAL)

- [ ] T157 [P] [US4] Contract test for GET /projects/{id}/runs/{id}/events in tests/contract/test_trace.py
- [ ] T158 [P] [US4] Contract test for WS /projects/{id}/runs/{id}/events/stream in tests/contract/test_trace.py
- [ ] T159 [P] [US4] Contract test for GET /projects/{id}/evidence in tests/contract/test_evidence.py
- [ ] T160 [P] [US4] Integration test for traceability in tests/integration/test_traceability.py (verify all workflows produce traces)

### Evidence & Trace Services (US4)
- [ ] T161 [P] [US4] Implement EvidenceService in memory/repository.py (CRUD, multi-entity linking, content hashing, reliability scoring)
- [ ] T162 [P] [US4] Implement TraceService in observability/events.py (AgentEvent persistence, chronological ordering, WebSocket streaming)
- [ ] T163 [P] [US4] Implement ReproducibilityService in research/comparison.py (experiment metadata capture: dataset version, code commit, model version, parameters, seed, environment, metrics, artifacts, timestamps per FR-005)

### API Routes (US4)
- [ ] T164 [P] [US4] Implement GET /projects/{id}/runs in apps/api/routes/runs.py (list agent runs with filters)
- [ ] T165 [P] [US4] Implement GET /projects/{id}/runs/{id} in apps/api/routes/runs.py (get run details)
- [ ] T166 [P] [US4] Implement GET /projects/{id}/runs/{id}/events in apps/api/routes/runs.py (paginated events)
- [ ] T167 [P] [US4] Implement WS /projects/{id}/runs/{id}/events/stream in apps/api/routes/runs.py (real-time event streaming)
- [ ] T168 [P] [US4] Implement POST /projects/{id}/runs/{id}/approval in apps/api/routes/runs.py (approval decisions)
- [ ] T169 [P] [US4] Implement GET /projects/{id}/evidence in apps/api/routes/evidence.py (list with filters)
- [ ] T170 [P] [US4] Implement GET /projects/{id}/evidence/{id} in apps/api/routes/evidence.py (get evidence details)

### CLI Commands (US4)
- [ ] T171 [P] [US4] Implement lab run list in cli/commands/run.py (list with filters)
- [ ] T172 [P] [US4] Implement lab run show in cli/commands/run.py (show run details)
- [ ] T173 [P] [US4] Implement lab run trace in cli/commands/run.py (trace with --follow for WebSocket)
- [ ] T174 [P] [US4] Implement lab run approve in cli/commands/run.py (approve/reject decisions)
- [ ] T175 [P] [US4] Implement lab evidence list in cli/commands/evidence.py (list with filters)
- [ ] T176 [P] [US4] Implement lab evidence show in cli/commands/evidence.py (show evidence details)

### Frontend (US4)
- [ ] T177 [P] [US4] Create Trace page in apps/web/src/pages/Trace.tsx (run selector, live trace viewer)
- [ ] T178 [P] [US4] Create EventTimeline component in apps/web/src/components/trace/EventTimeline.tsx (chronological events with expandable tool I/O)
- [ ] T179 [P] [US4] Create EvidencePanel component in apps/web/src/components/trace/EvidencePanel.tsx (evidence list with previews, links to source)
- [ ] T180 [P] [US4] Create ClaimTypeBadges component in apps/web/src/components/common/ClaimTypeBadges.tsx (Observed/Inferred/Recommended/Not Tested/Unknown badges per FR-019)
- [ ] T181 [P] [US4] Create ReproducibilityPanel component in apps/web/src/components/research/ReproducibilityPanel.tsx (experiment metadata display)
- [ ] T182 [US4] Add trace/evidence API methods to apps/web/src/services/api.ts

**Checkpoint**: At this point, all 4 P1 user stories should work independently

---

## Phase 7: User Story 5 - Human-in-the-Loop Approval (Priority: P2)

**Goal**: High-impact actions require explicit human approval with visible action details, risk assessment, recorded decisions.

**Independent Test**: Configure action requiring approval → verify system pauses, presents approval UI with details/risk, records decision.

### Tests for User Story 5 (OPTIONAL)

- [ ] T183 [P] [US5] Contract test for POST /projects/{id}/runs/{id}/approval in tests/contract/test_approval.py
- [ ] T184 [P] [US5] Integration test for approval workflow in tests/integration/test_approval_workflow.py

### Approval Infrastructure (US5)
- [ ] T185 [P] [US5] Implement ApprovalPolicy in agent_runtime/policies.py (high-impact actions: production_deployment, destructive_filesystem, external_pr, network_access, sensitive_data, budget_exceeded, security_policy_change per FR-020)
- [ ] T186 [P] [US5] Implement ApprovalManager in agent_runtime/policies.py (request approval, present details/risk, record decision, enforce fail-closed on rejection)
- [ ] T187 [P] [US5] Add approval integration to AgentRun model (awaiting_approval status, approval workflow)

### API Routes (US5)
- [ ] T188 [P] [US5] Implement GET /projects/{id}/runs/{id}/approval/pending in apps/api/routes/runs.py (list pending approvals)
- [ ] T189 [P] [US5] WebSocket already covered in T167 for real-time approval notifications

### CLI Commands (US5)
- [ ] T190 [P] [US5] lab run approve already covered in T174

### Frontend (US5)
- [ ] T191 [P] [US5] Create ApprovalDialog component in apps/web/src/components/common/ApprovalDialog.tsx (action details, risk level, approve/reject buttons)
- [ ] T192 [P] [US5] Create ApprovalHistory component in apps/web/src/components/trace/ApprovalHistory.tsx (list of approval requests with decisions)
- [ ] T193 [US5] Integrate approval WebSocket notifications in apps/web/src/hooks/useApprovals.ts

**Checkpoint**: User Story 5 functional, integrates with all previous stories

---

## Phase 8: User Story 6 - Model-Agnostic Agent Execution (Priority: P2)

**Goal**: Agents operate against common model interface supporting OpenAI, Anthropic, Gemini, Ollama, vLLM, MLX. Model selection configurable without code changes.

**Independent Test**: Configure different providers → verify agents execute successfully without code changes.

### Tests for User Story 6 (OPTIONAL)

- [ ] T194 [P] [US6] Contract test for GET /config in tests/contract/test_config.py (verify model providers listed)
- [ ] T195 [P] [US6] Integration test for model switching in tests/integration/test_model_agnostic.py (test with 3+ providers)

### Model Abstraction (US6) - Already partially done in T039, T040
- [ ] T196 [P] [US6] Add provider-specific implementations in tools/llm.py (OpenAIClient, AnthropicClient, GeminiClient, OllamaClient, VLLMClient, MLXClient)
- [ ] T197 [P] [US6] Implement model capability registry in agent_runtime/router.py (context window, max tokens, supported features per model)
- [ ] T198 [P] [US6] Add cost/latency tracking per provider in observability/metrics.py (for leaderboard per FR-023, SC-010)

### API Routes (US6)
- [ ] T199 [P] [US6] GET /config already covered in T043 (returns llm_providers list)
- [ ] T200 [P] [US6] Implement GET /models/leaderboard in apps/api/routes/models.py (model performance comparison per contracts/api.md)

### CLI Commands (US6)
- [ ] T201 [P] [US6] Implement lab models leaderboard in cli/commands/models.py (show leaderboard)
- [ ] T202 [P] [US6] Implement lab config show/set/validate in cli/commands/config.py (configuration management)

### Frontend (US6)
- [ ] T203 [P] [US6] Create Models page in apps/web/src/pages/Models.tsx (provider selection, model leaderboard, cost/latency charts)
- [ ] T204 [P] [US6] Create ModelSelector component in apps/web/src/components/common/ModelSelector.tsx (provider/model dropdown with capabilities)
- [ ] T205 [US6] Add models API methods to apps/web/src/services/api.ts

**Checkpoint**: All 6 user stories functional independently

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

### Dashboard & Navigation
- [ ] T206 [P] Create Dashboard page in apps/web/src/pages/Dashboard.tsx (active runs summary, recent experiments, governance assessments, red team runs, quick actions)
- [ ] T207 [P] Create Navigation component in apps/web/src/components/common/Navigation.tsx (sidebar with links to all pages)
- [ ] T208 [P] Create Settings page in apps/web/src/pages/Settings.tsx (LLM config, approval policies, sandbox settings, cost limits)

### API & Infrastructure
- [ ] T209 [P] Implement GET /health in apps/api/main.py (health check with DB status)
- [ ] T210 [P] Implement GET /config in apps/api/main.py (system configuration)
- [ ] T211 [P] Add authentication middleware in apps/api/middleware/auth.py (API key validation)
- [ ] T212 [P] Add request logging middleware in apps/api/middleware/logging.py (structured request/response logs)
- [ ] T213 [P] Add error handling middleware in apps/api/middleware/errors.py (map exceptions to error codes per contracts/api.md)
- [ ] T214 [P] Add CORS middleware in apps/api/middleware/cors.py (configurable origins)
- [ ] T215 [P] Implement API versioning in apps/api/main.py (v1 prefix, deprecation strategy)

### Testing & Quality
- [ ] T216 [P] Create unit tests for core utilities in tests/unit/ (config, logging, events, exceptions)
- [ ] T217 [P] Create unit tests for tools in tests/unit/tools/ (profiling, training, evaluation, security)
- [ ] T218 [P] Create unit tests for agents in tests/unit/agents/ (base agent, each agent type)
- [ ] T219 [P] Create unit tests for research in tests/unit/research/ (hypotheses, planner, experiments, metrics, memory)
- [ ] T220 [P] Create unit tests for governance in tests/unit/governance/ (assessment, risk, controls, scoring)
- [ ] T221 [P] Create unit tests for redteam in tests/unit/redteam/ (planner, attacks, evaluator)
- [ ] T222 [P] Create unit tests for security in tests/unit/security/ (sandbox, command_guard, secrets, prompt_injection)
- [ ] T223 [P] Create unit tests for memory in tests/unit/memory/ (models, repository, retrieval)
- [ ] T224 [P] Create unit tests for storage in tests/unit/storage/ (database, artifacts)
- [ ] T225 [P] Create unit tests for observability in tests/unit/observability/ (tracing, metrics, events)
- [ ] T226 [P] Create unit tests for CLI in tests/unit/cli/ (all commands)
- [ ] T227 [P] Create security tests in tests/security/ (sandbox escape attempts, command injection, secret leakage, prompt injection)
- [ ] T228 [P] Configure pytest.ini with asyncio, coverage, markers
- [ ] T229 [P] Configure Vitest for frontend in apps/web/vitest.config.ts
- [ ] T230 [P] Create GitHub Actions CI workflow (.github/workflows/ci.yml: ruff, mypy, pytest, security tests, Docker build)

### Documentation & Examples
- [ ] T231 [P] Create synthetic fraud dataset in data/demo/fraud.csv (4.8M rows, 147 columns per quickstart.md)
- [ ] T232 [P] Create cosmetics recommendation system demo in examples/recommendation_system/system.yaml
- [ ] T233 [P] Create vulnerable chatbot demo in examples/llm_chatbot/target.yaml
- [ ] T234 [P] Create broken ML project demo in examples/broken_ml_project/
- [ ] T235 [P] Write architecture documentation in docs/architecture.md
- [ ] T236 [P] Write API documentation in docs/api.md (from contracts/api.md)
- [ ] T237 [P] Write CLI documentation in docs/cli.md (from contracts/cli.md)
- [ ] T238 [P] Write agent development guide in docs/agents.md
- [ ] T239 [P] Write tool development guide in docs/tools.md

### Validation
- [ ] T240 Run quickstart.md Scenario 1: Fraud Research Demo validation
- [ ] T241 Run quickstart.md Scenario 2: Governance Demo validation
- [ ] T242 Run quickstart.md Scenario 3: Red Team Demo validation
- [ ] T243 Run quickstart.md Scenario 4: Human Approval validation
- [ ] T244 Run quickstart.md Scenario 5: Model Agnostic validation
- [ ] T245 Run quickstart.md Scenario 6: Security Controls validation
- [ ] T246 Run quickstart.md Scenario 7: Cost Tracking validation
- [ ] T247 Run quickstart.md Scenario 8: Quality Gates validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-8)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2)
- **Polish (Phase 9)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P1)**: Can start after Foundational (Phase 2) - May integrate with US1 but independently testable
- **User Story 3 (P1)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but independently testable
- **User Story 4 (P1)**: Can start after Foundational (Phase 2) - Cross-cutting, enhances all stories
- **User Story 5 (P2)**: Can start after Foundational (Phase 2) - Integrates with all stories via approval workflow
- **User Story 6 (P2)**: Can start after Foundational (Phase 2) - Integrates with all stories via model abstraction

### Within Each User Story

- Tests (if included) MUST be written and FAIL before implementation
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all tests for User Story 1 together (if tests requested):
Task: "Contract test for POST /projects/{id}/research in tests/contract/test_research.py"
Task: "Contract test for GET /projects/{id}/research/{id}/report in tests/contract/test_research.py"
Task: "Contract test for WS /projects/{id}/runs/{id}/events/stream in tests/contract/test_research.py"
Task: "Integration test for full research workflow in tests/integration/test_research_workflow.py"

# Launch all models for User Story 1 together:
Task: "Create ResearchQuestion repository in storage/repository/research_question.py"
Task: "Create Hypothesis repository in storage/repository/hypothesis.py"
Task: "Create Experiment repository in storage/repository/experiment.py"
Task: "Create Metric repository in storage/repository/metric.py"
Task: "Create ExperimentArtifact repository in storage/repository/experiment_artifact.py"

# Launch all tools for User Story 1 together:
Task: "Implement dataset profiling tool in tools/profiling.py"
Task: "Implement Python execution tool in tools/python.py"
Task: "Implement shell execution tool in tools/shell.py"
Task: "Implement training tool in tools/training.py"
Task: "Implement evaluation tool in tools/evaluation.py"
Task: "Implement benchmarking tool in tools/benchmarking.py"

# Launch all research agents for User Story 1 together:
Task: "Implement DatasetAnalyst agent in agents/research/dataset_analyst.py"
Task: "Implement ResearchPlanner agent in agents/research/research_planner.py"
Task: "Implement ExperimentPlanner agent in agents/research/experiment_planner.py"
Task: "Implement ExperimentExecutor agent in agents/research/experiment_executor.py"
Task: "Implement Evaluator agent in agents/research/evaluator.py"
Task: "Implement Critic agent in agents/critic.py"
Task: "Implement ResearchDirector agent in agents/director.py"
Task: "Implement ResearchMemory in memory/research_memory.py"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently (run quickstart.md Scenario 1)
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Add User Story 4 → Test independently → Deploy/Demo
6. Add User Story 5 → Test independently → Deploy/Demo
7. Add User Story 6 → Test independently → Deploy/Demo
8. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (Research)
   - Developer B: User Story 2 (Governance)
   - Developer C: User Story 3 (Red Team)
   - Developer D: User Story 4 (Traceability - cross-cutting)
3. Stories complete and integrate independently
4. Then: User Story 5 (Approval) + User Story 6 (Model Agnostic)

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
- Constitutional principles enforced: Evidence Over Claims (T053, T063, T064), Agents Decide/Tools Execute (T053-T058, T059-T065), Reproducibility (T014, T163), Security by Default (T034-T038), Human Oversight (T185-T193), Model Agnosticism (T039, T040, T196-T198), Explicit State (T024, T162), Bounded Autonomy (T067, T114, T142), Fail Safely (T034, T186), Testability (T216-T227), Observability (T041-T043), Simplicity (single project structure, PostgreSQL, Docker)