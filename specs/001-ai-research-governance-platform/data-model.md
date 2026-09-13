# Data Model: AI Research & Governance Platform

**Feature**: AI Research & Governance Platform
**Date**: 2026-09-12
**Status**: Complete

## Entity Relationship Overview

```
Project (1) ─────< (N) ResearchQuestion
Project (1) ─────< (N) GovernanceAssessment
Project (1) ─────< (N) RedTeamRun

ResearchQuestion (1) ─────< (N) Hypothesis
Hypothesis (1) ─────< (N) Experiment
Experiment (1) ─────< (N) Metric
Experiment (1) ─────< (N) ExperimentArtifact

GovernanceAssessment (1) ─────< (N) Finding
GovernanceAssessment (1) ─────< (N) Evidence
GovernanceControl (N) ─────< (M) Finding (via control_id)

RedTeamRun (1) ─────< (N) RedTeamTest
RedTeamTest (1) ─────< (N) Evidence

AgentRun (1) ─────< (N) AgentEvent
AgentRun (1) ─────< (N) Decision
AgentRun (N) ─────< (M) Experiment (via experiment_id)
AgentRun (N) ─────< (M) GovernanceAssessment (via assessment_id)
AgentRun (N) ─────< (M) RedTeamRun (via redteam_run_id)
```

## Core Entities

### Project
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| name | String | NOT NULL, max 255 | Human-readable name |
| description | Text | NULLABLE | Detailed description |
| repository_url | String | NULLABLE, max 500 | Git repository URL |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### ResearchQuestion
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| project_id | UUID | FK → Project.id, NOT NULL | Parent project |
| question | Text | NOT NULL | Research question text |
| objective | Text | NOT NULL | Specific objective |
| constraints | JSONB | NOT NULL, default '{}' | Constraints (e.g., precision >= 0.95) |
| success_criteria | JSONB | NOT NULL, default '{}' | Measurable success criteria |
| status | Enum | NOT NULL, default 'pending' | pending, running, completed, failed, cancelled |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### Hypothesis
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| research_question_id | UUID | FK → ResearchQuestion.id, NOT NULL | Parent research question |
| statement | Text | NOT NULL | Hypothesis statement |
| rationale | Text | NULLABLE | Reasoning behind hypothesis |
| expected_outcome | JSONB | NOT NULL, default '{}' | Expected metrics/results |
| confidence | Float | NOT NULL, default 0.5 | Confidence 0.0-1.0 |
| status | Enum | NOT NULL, default 'proposed' | proposed, testing, accepted, rejected, inconclusive |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### Experiment
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| hypothesis_id | UUID | FK → Hypothesis.id, NOT NULL | Parent hypothesis |
| name | String | NOT NULL, max 255 | Experiment name |
| description | Text | NULLABLE | Detailed description |
| dataset_version | String | NOT NULL, max 255 | Dataset version/hash |
| code_commit | String | NOT NULL, max 64 | Git commit SHA |
| model_name | String | NOT NULL, max 255 | Model identifier |
| model_version | String | NULLABLE, max 255 | Model version |
| parameters | JSONB | NOT NULL, default '{}' | Hyperparameters |
| seed | Integer | NULLABLE | Random seed |
| status | Enum | NOT NULL, default 'pending' | pending, running, completed, failed, cancelled |
| started_at | DateTime | NULLABLE | Execution start |
| completed_at | DateTime | NULLABLE | Execution end |
| duration_seconds | Float | NULLABLE | Execution duration |
| cost_usd | Float | NULLABLE | Estimated cost |
| error_message | Text | NULLABLE | Error if failed |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### Metric
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| experiment_id | UUID | FK → Experiment.id, NOT NULL | Parent experiment |
| name | String | NOT NULL, max 100 | Metric name (e.g., precision, recall) |
| value | Float | NOT NULL | Metric value |
| unit | String | NULLABLE, max 50 | Unit (%, seconds, etc.) |
| direction | Enum | NOT NULL | higher_better, lower_better, target |
| baseline_value | Float | NULLABLE | Baseline for comparison |
| delta | Float | NULLABLE | Difference from baseline |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |

### ExperimentArtifact
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| experiment_id | UUID | FK → Experiment.id, NOT NULL | Parent experiment |
| artifact_type | Enum | NOT NULL | model, plot, log, dataset, report, other |
| path | String | NOT NULL, max 1000 | Storage path |
| checksum | String | NOT NULL, max 64 | SHA256 checksum |
| size_bytes | Integer | NOT NULL | File size |
| metadata | JSONB | NOT NULL, default '{}' | Additional metadata |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |

### GovernanceAssessment
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| project_id | UUID | FK → Project.id, NOT NULL | Parent project |
| system_name | String | NOT NULL, max 255 | AI system name |
| system_description | Text | NULLABLE | System description |
| architecture | JSONB | NOT NULL, default '{}' | System architecture |
| data_inventory | JSONB | NOT NULL, default '{}' | Data inventory |
| models | JSONB | NOT NULL, default '[]' | Model inventory |
| vendors | JSONB | NOT NULL, default '[]' | Vendor list |
| deployment_regions | JSONB | NOT NULL, default '[]' | Deployment regions |
| user_populations | JSONB | NOT NULL, default '[]' | User populations |
| intended_use | Text | NULLABLE | Intended use |
| prohibited_use | Text | NULLABLE | Prohibited use |
| existing_controls | JSONB | NOT NULL, default '[]' | Existing controls |
| framework | String | NOT NULL, max 100 | Framework used (nist_ai_rmf, iso_42001, enterprise_ai) |
| framework_version | String | NOT NULL, max 50 | Framework version |
| risk_level | Enum | NOT NULL | critical, high, medium, low, informational |
| risk_score | Float | NOT NULL | Overall risk score 0-100 |
| risk_breakdown | JSONB | NOT NULL, default '{}' | Per-category scores |
| status | Enum | NOT NULL, default 'pending' | pending, running, completed, failed |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### GovernanceControl
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| framework | String | NOT NULL, max 100 | Framework name |
| framework_version | String | NOT NULL, max 50 | Framework version |
| control_id | String | NOT NULL, max 100 | Stable control identifier (e.g., AI-DATA-001) |
| title | String | NOT NULL, max 255 | Control title |
| description | Text | NOT NULL | Control description |
| severity | Enum | NOT NULL | critical, high, medium, low, informational |
| category | String | NOT NULL, max 100 | Risk category |
| requirements | JSONB | NOT NULL, default '[]' | Requirements list |
| evidence_types | JSONB | NOT NULL, default '[]' | Required evidence types |
| pass_condition | Text | NOT NULL | Pass condition description |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |
| **Unique** | | (framework, framework_version, control_id) | |

### Finding
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| assessment_id | UUID | FK → GovernanceAssessment.id, NOT NULL | Parent assessment |
| control_id | UUID | FK → GovernanceControl.id, NULLABLE | Related control |
| severity | Enum | NOT NULL | critical, high, medium, low, informational |
| category | String | NOT NULL, max 100 | Finding category |
| title | String | NOT NULL, max 255 | Finding title |
| description | Text | NOT NULL | Detailed description |
| recommendation | Text | NULLABLE | Remediation recommendation |
| status | Enum | NOT NULL, default 'open' | open, in_progress, resolved, accepted_risk, false_positive |
| architecture_component | String | NULLABLE, max 255 | Affected component |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### Evidence
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| source_type | Enum | NOT NULL | tool_output, document, code, dataset, model, manual, external |
| source_reference | String | NOT NULL, max 500 | Reference to source (file path, URL, tool name) |
| content_hash | String | NOT NULL, max 64 | SHA256 of content |
| content_preview | Text | NULLABLE | First 500 chars for display |
| collected_at | DateTime | NOT NULL, default now() | Collection timestamp |
| reliability | Enum | NOT NULL | high, medium, low, unknown |
| related_control_id | UUID | FK → GovernanceControl.id, NULLABLE | Related control |
| related_finding_id | UUID | FK → Finding.id, NULLABLE | Related finding |
| related_experiment_id | UUID | FK → Experiment.id, NULLABLE | Related experiment |
| related_redteam_test_id | UUID | FK → RedTeamTest.id, NULLABLE | Related red team test |
| metadata | JSONB | NOT NULL, default '{}' | Additional metadata |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |

### RedTeamRun
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| project_id | UUID | FK → Project.id, NOT NULL | Parent project |
| target_system | String | NOT NULL, max 255 | Target system name |
| target_description | Text | NULLABLE | Target description |
| target_type | Enum | NOT NULL | llm_app, agent, api, prompt, tool, data_flow |
| status | Enum | NOT NULL, default 'pending' | pending, running, completed, failed |
| started_at | DateTime | NULLABLE | Run start |
| completed_at | DateTime | NULLABLE | Run end |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### RedTeamTest
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| redteam_run_id | UUID | FK → RedTeamRun.id, NOT NULL | Parent run |
| scenario | String | NOT NULL, max 255 | Attack scenario name |
| attack_type | Enum | NOT NULL | prompt_injection, tool_abuse, secret_exposure, command_injection, path_traversal, data_poisoning, hallucination, unsafe_recommendation, context_manipulation, goal_hijacking, system_prompt_extraction, sensitive_disclosure, unauthorized_tool, excessive_permissions, indirect_injection |
| input | Text | NOT NULL | Attack input |
| expected_behavior | Text | NOT NULL | Expected safe behavior |
| actual_behavior | Text | NULLABLE | Actual system behavior |
| result | Enum | NOT NULL | pass, fail, inconclusive, error |
| severity | Enum | NOT NULL | critical, high, medium, low, informational |
| evidence_ids | UUID[] | NOT NULL, default '[]' | Linked evidence |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### AgentRun
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| agent_type | Enum | NOT NULL | research_director, dataset_analyst, research_planner, experiment_planner, experiment_executor, evaluator, critic, governance_analyst, redteam_planner, redteam_executor |
| objective | Text | NOT NULL | Run objective |
| status | Enum | NOT NULL, default 'pending' | pending, running, completed, failed, cancelled, awaiting_approval |
| parent_run_id | UUID | FK → AgentRun.id, NULLABLE | Parent run (for sub-agents) |
| project_id | UUID | FK → Project.id, NOT NULL | Parent project |
| research_question_id | UUID | FK → ResearchQuestion.id, NULLABLE | Related research question |
| hypothesis_id | UUID | FK → Hypothesis.id, NULLABLE | Related hypothesis |
| experiment_id | UUID | FK → Experiment.id, NULLABLE | Related experiment |
| governance_assessment_id | UUID | FK → GovernanceAssessment.id, NULLABLE | Related assessment |
| redteam_run_id | UUID | FK → RedTeamRun.id, NULLABLE | Related red team run |
| started_at | DateTime | NULLABLE | Run start |
| completed_at | DateTime | NULLABLE | Run end |
| total_tokens | Integer | NULLABLE | Total tokens used |
| estimated_cost_usd | Float | NULLABLE | Estimated cost |
| error_message | Text | NULLABLE | Error if failed |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

### AgentEvent
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| agent_run_id | UUID | FK → AgentRun.id, NOT NULL | Parent run |
| timestamp | DateTime | NOT NULL, default now() | Event timestamp |
| event_type | Enum | NOT NULL | planning, tool_call, tool_result, decision, error, approval_request, approval_response, evaluation, critique, memory_store, memory_retrieve |
| message | Text | NULLABLE | Human-readable message |
| tool_name | String | NULLABLE, max 100 | Tool invoked |
| tool_input | JSONB | NULLABLE | Tool input arguments |
| tool_output | JSONB | NULLABLE | Tool output result |
| status | Enum | NOT NULL | success, failure, partial, pending |
| duration_ms | Integer | NULLABLE | Duration in milliseconds |
| evidence_id | UUID | FK → Evidence.id, NULLABLE | Associated evidence |
| metadata | JSONB | NOT NULL, default '{}' | Additional metadata |

### Decision
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| agent_run_id | UUID | FK → AgentRun.id, NOT NULL | Parent run |
| decision_type | Enum | NOT NULL | continue, modify_hypothesis, new_hypothesis, stop, approve, reject, request_human |
| decision | Text | NOT NULL | Decision description |
| rationale | Text | NOT NULL | Reasoning |
| evidence_ids | UUID[] | NOT NULL, default '[]' | Supporting evidence |
| confidence | Float | NOT NULL, default 0.5 | Confidence 0.0-1.0 |
| requires_human_approval | Boolean | NOT NULL, default false | Human approval required |
| approval_status | Enum | NULLABLE | pending, approved, rejected |
| approved_by | String | NULLABLE, max 255 | Approver identifier |
| approved_at | DateTime | NULLABLE | Approval timestamp |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |

## Research Memory Entities

### ResearchMemory
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, default gen_random_uuid() | Unique identifier |
| project_id | UUID | FK → Project.id, NOT NULL | Parent project |
| statement | Text | NOT NULL | Memory statement |
| confidence | Float | NOT NULL, default 0.5 | Confidence 0.0-1.0 |
| evidence_ids | UUID[] | NOT NULL, default '[]' | Supporting evidence |
| tags | String[] | NOT NULL, default '[]' | Search tags |
| source_type | Enum | NOT NULL | hypothesis, experiment, failure, conclusion, lesson |
| source_id | UUID | NULLABLE | Source entity ID |
| created_at | DateTime | NOT NULL, default now() | Creation timestamp |
| updated_at | DateTime | NOT NULL, default now() | Last update timestamp |

## Enums Summary

- **ResearchQuestionStatus**: pending, running, completed, failed, cancelled
- **HypothesisStatus**: proposed, testing, accepted, rejected, inconclusive
- **ExperimentStatus**: pending, running, completed, failed, cancelled
- **MetricDirection**: higher_better, lower_better, target
- **ArtifactType**: model, plot, log, dataset, report, other
- **GovernanceStatus**: pending, running, completed, failed
- **RiskLevel**: critical, high, medium, low, informational
- **FindingStatus**: open, in_progress, resolved, accepted_risk, false_positive
- **EvidenceSourceType**: tool_output, document, code, dataset, model, manual, external
- **EvidenceReliability**: high, medium, low, unknown
- **RedTeamRunStatus**: pending, running, completed, failed
- **RedTeamAttackType**: prompt_injection, tool_abuse, secret_exposure, command_injection, path_traversal, data_poisoning, hallucination, unsafe_recommendation, context_manipulation, goal_hijacking, system_prompt_extraction, sensitive_disclosure, unauthorized_tool, excessive_permissions, indirect_injection
- **RedTeamResult**: pass, fail, inconclusive, error
- **AgentType**: research_director, dataset_analyst, research_planner, experiment_planner, experiment_executor, evaluator, critic, governance_analyst, redteam_planner, redteam_executor
- **AgentRunStatus**: pending, running, completed, failed, cancelled, awaiting_approval
- **EventType**: planning, tool_call, tool_result, decision, error, approval_request, approval_response, evaluation, critique, memory_store, memory_retrieve
- **DecisionType**: continue, modify_hypothesis, new_hypothesis, stop, approve, reject, request_human
- **ApprovalStatus**: pending, approved, rejected
- **MemorySourceType**: hypothesis, experiment, failure, conclusion, lesson

## Indexes

```sql
-- Performance indexes
CREATE INDEX idx_research_question_project ON research_question(project_id);
CREATE INDEX idx_hypothesis_research_question ON hypothesis(research_question_id);
CREATE INDEX idx_experiment_hypothesis ON experiment(hypothesis_id);
CREATE INDEX idx_experiment_status ON experiment(status);
CREATE INDEX idx_metric_experiment ON metric(experiment_id);
CREATE INDEX idx_artifact_experiment ON experiment_artifact(experiment_id);
CREATE INDEX idx_governance_assessment_project ON governance_assessment(project_id);
CREATE INDEX idx_finding_assessment ON finding(assessment_id);
CREATE INDEX idx_finding_control ON finding(control_id);
CREATE INDEX idx_evidence_control ON evidence(related_control_id);
CREATE INDEX idx_evidence_finding ON evidence(related_finding_id);
CREATE INDEX idx_evidence_experiment ON evidence(related_experiment_id);
CREATE INDEX idx_redteam_run_project ON redteam_run(project_id);
CREATE INDEX idx_redteam_test_run ON redteam_test(redteam_run_id);
CREATE INDEX idx_agent_run_project ON agent_run(project_id);
CREATE INDEX idx_agent_run_parent ON agent_run(parent_run_id);
CREATE INDEX idx_agent_event_run ON agent_event(agent_run_id);
CREATE INDEX idx_agent_event_timestamp ON agent_event(timestamp);
CREATE INDEX idx_decision_run ON decision(agent_run_id);
CREATE INDEX idx_research_memory_project ON research_memory(project_id);
CREATE INDEX idx_research_memory_tags ON research_memory USING GIN(tags);
```

## Validation Rules

1. **Experiment reproducibility**: dataset_version, code_commit, model_name, model_version, parameters, seed MUST be set before experiment runs
2. **Evidence integrity**: content_hash MUST match actual content on retrieval
3. **Governance traceability**: Every finding MUST link to at least one control and one evidence
4. **Agent traceability**: Every agent run MUST produce at least one AgentEvent
5. **Cost tracking**: estimated_cost_usd MUST be calculated for completed agent runs
6. **Approval workflow**: Decisions with requires_human_approval=true MUST have approval_status set before execution continues
7. **Memory consistency**: ResearchMemory evidence_ids MUST reference existing Evidence records