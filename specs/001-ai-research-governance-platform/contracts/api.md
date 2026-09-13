# API Contracts: AI Research & Governance Platform

**Feature**: AI Research & Governance Platform
**Date**: 2026-09-12
**Status**: Complete

## Overview

The platform exposes a FastAPI REST API for all core operations. All endpoints return JSON. Authentication is via API key header for MVP (single-user). WebSocket/SSE endpoints provide real-time agent trace updates.

Base URL: `http://localhost:8000/api/v1`

## Authentication

```
Authorization: Bearer <API_KEY>
X-API-Key: <API_KEY>
```

## Common Response Formats

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "meta": { ... }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": { ... }
  }
}
```

### Paginated Response
```json
{
  "success": true,
  "data": [...],
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

## Project Endpoints

### Create Project
```
POST /projects
```

**Request:**
```json
{
  "name": "string",
  "description": "string (optional)",
  "repository_url": "string (optional)"
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "repository_url": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
}
```

### List Projects
```
GET /projects?page=1&page_size=20
```

**Response:** 200 OK (paginated)

### Get Project
```
GET /projects/{project_id}
```

**Response:** 200 OK

### Update Project
```
PATCH /projects/{project_id}
```

**Request:**
```json
{
  "name": "string (optional)",
  "description": "string (optional)",
  "repository_url": "string (optional)"
}
```

### Delete Project
```
DELETE /projects/{project_id}
```

**Response:** 204 No Content

## Research Endpoints

### Create Research Question
```
POST /projects/{project_id}/research
```

**Request:**
```json
{
  "question": "string",
  "objective": "string",
  "constraints": { "precision": ">=0.95" },
  "success_criteria": { "min_recall_improvement": 0.1 },
  "max_experiments": 10,
  "budget_usd": 5.0
}
```

**Response:** 201 Created
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "project_id": "uuid",
    "question": "string",
    "objective": "string",
    "constraints": {},
    "success_criteria": {},
    "status": "pending",
    "created_at": "datetime"
  }
}
```

### Get Research Question
```
GET /projects/{project_id}/research/{research_id}
```

### List Research Questions
```
GET /projects/{project_id}/research?status=completed&page=1&page_size=20
```

### Start Research Run
```
POST /projects/{project_id}/research/{research_id}/run
```

**Response:** 202 Accepted
```json
{
  "success": true,
  "data": {
    "agent_run_id": "uuid",
    "status": "running"
  }
}
```

### Get Research Status
```
GET /projects/{project_id}/research/{research_id}/status
```

**Response:**
```json
{
  "success": true,
  "data": {
    "research_id": "uuid",
    "status": "running",
    "current_phase": "experimenting",
    "experiments_completed": 3,
    "experiments_total": 10,
    "latest_agent_run_id": "uuid"
  }
}
```

### Get Research Report
```
GET /projects/{project_id}/research/{research_id}/report
```

**Response:**
```json
{
  "success": true,
  "data": {
    "research_id": "uuid",
    "executive_summary": "string",
    "objective": "string",
    "methodology": "string",
    "experiments": [...],
    "results": {...},
    "evidence": [...],
    "failures": [...],
    "limitations": [...],
    "recommendations": [...],
    "reproducibility": {...},
    "agent_trace_summary": [...],
    "claim_types": {
      "observed": [...],
      "inferred": [...],
      "recommended": [...],
      "not_tested": [...],
      "unknown": [...]
    }
  }
}
```

## Experiment Endpoints

### List Experiments
```
GET /projects/{project_id}/experiments?research_question_id={id}&status=completed&page=1&page_size=20
```

### Get Experiment
```
GET /projects/{project_id}/experiments/{experiment_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "hypothesis_id": "uuid",
    "name": "string",
    "dataset_version": "string",
    "code_commit": "string",
    "model_name": "string",
    "model_version": "string",
    "parameters": {},
    "seed": 42,
    "status": "completed",
    "metrics": [
      {"name": "precision", "value": 0.953, "unit": "", "direction": "higher_better", "baseline_value": 0.961, "delta": -0.008},
      {"name": "recall", "value": 0.849, "unit": "", "direction": "higher_better", "baseline_value": 0.714, "delta": 0.135}
    ],
    "artifacts": [...],
    "started_at": "datetime",
    "completed_at": "datetime",
    "duration_seconds": 120.5,
    "cost_usd": 0.02
  }
}
```

### Get Experiment Artifacts
```
GET /projects/{project_id}/experiments/{experiment_id}/artifacts
```

### Download Artifact
```
GET /projects/{project_id}/experiments/{experiment_id}/artifacts/{artifact_id}/download
```

## Hypothesis Endpoints

### List Hypotheses
```
GET /projects/{project_id}/hypotheses?research_question_id={id}&status=accepted
```

### Get Hypothesis
```
GET /projects/{project_id}/hypotheses/{hypothesis_id}
```

## Governance Endpoints

### Create Governance Assessment
```
POST /projects/{project_id}/governance
```

**Request:**
```json
{
  "system_name": "string",
  "system_description": "string",
  "architecture": {},
  "data_inventory": {},
  "models": [],
  "vendors": [],
  "deployment_regions": [],
  "user_populations": [],
  "intended_use": "string",
  "prohibited_use": "string",
  "existing_controls": [],
  "framework": "nist_ai_rmf",
  "framework_version": "1.0"
}
```

**Response:** 201 Created

### Get Governance Assessment
```
GET /projects/{project_id}/governance/{assessment_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "system_name": "string",
    "framework": "nist_ai_rmf",
    "framework_version": "1.0",
    "risk_level": "high",
    "risk_score": 67.0,
    "risk_breakdown": {
      "privacy": 72,
      "security": 61,
      "fairness": 54,
      "reliability": 41,
      "governance": 78
    },
    "findings": [...],
    "evidence": [...],
    "remediation_plan": [...],
    "status": "completed"
  }
}
```

### List Governance Assessments
```
GET /projects/{project_id}/governance?status=completed&page=1&page_size=20
```

### Get Governance Frameworks
```
GET /governance/frameworks
```

**Response:**
```json
{
  "success": true,
  "data": [
    {"id": "nist_ai_rmf", "name": "NIST AI RMF", "version": "1.0", "controls_count": 42},
    {"id": "iso_42001", "name": "ISO 42001", "version": "2023", "controls_count": 38},
    {"id": "enterprise_ai", "name": "Enterprise AI Policy", "version": "1.0", "controls_count": 25}
  ]
}
```

### Get Framework Controls
```
GET /governance/frameworks/{framework_id}/controls?version=1.0
```

## Red Team Endpoints

### Create Red Team Run
```
POST /projects/{project_id}/redteam
```

**Request:**
```json
{
  "target_system": "string",
  "target_description": "string",
  "target_type": "llm_app",
  "scenarios": ["prompt_injection", "tool_abuse", "secret_exposure"]
}
```

**Response:** 201 Created

### Get Red Team Run
```
GET /projects/{project_id}/redteam/{run_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "target_system": "string",
    "target_type": "llm_app",
    "status": "completed",
    "tests": [
      {
        "id": "uuid",
        "scenario": "prompt_injection",
        "attack_type": "prompt_injection",
        "input": "string",
        "expected_behavior": "string",
        "actual_behavior": "string",
        "result": "fail",
        "severity": "high",
        "evidence_ids": ["uuid"]
      }
    ],
    "summary": {
      "total": 15,
      "pass": 10,
      "fail": 3,
      "inconclusive": 2,
      "critical": 0,
      "high": 2,
      "medium": 1,
      "low": 0
    }
  }
}
```

### List Red Team Runs
```
GET /projects/{project_id}/redteam?status=completed&page=1&page_size=20
```

### Get Attack Scenarios
```
GET /redteam/scenarios
```

**Response:**
```json
{
  "success": true,
  "data": [
    {"id": "prompt_injection", "name": "Prompt Injection", "description": "..."},
    {"id": "tool_abuse", "name": "Tool Abuse", "description": "..."},
    ...
  ]
}
```

## Agent Run Endpoints

### List Agent Runs
```
GET /projects/{project_id}/runs?agent_type=research_director&status=completed&page=1&page_size=20
```

### Get Agent Run
```
GET /projects/{project_id}/runs/{run_id}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "agent_type": "research_director",
    "objective": "string",
    "status": "completed",
    "started_at": "datetime",
    "completed_at": "datetime",
    "total_tokens": 15000,
    "estimated_cost_usd": 0.45,
    "events_count": 42
  }
}
```

### Get Agent Run Events (Trace)
```
GET /projects/{project_id}/runs/{run_id}/events?page=1&page_size=100
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "timestamp": "datetime",
      "event_type": "tool_call",
      "message": "Profiling dataset",
      "tool_name": "profile_dataset",
      "tool_input": {"dataset_id": "uuid"},
      "tool_output": {"rows": 4800000, "columns": 147, ...},
      "status": "success",
      "duration_ms": 1250,
      "evidence_id": "uuid"
    }
  ],
  "meta": {...}
}
```

### Stream Agent Run Events (WebSocket)
```
WS /projects/{project_id}/runs/{run_id}/events/stream
```

**Messages:**
```json
// Server -> Client
{"type": "event", "data": {...AgentEvent...}}
{"type": "status", "data": {"status": "running", "current_phase": "experimenting"}}
{"type": "complete", "data": {"status": "completed"}}
{"type": "error", "data": {"error": "..."}}
```

### Request Approval
```
POST /projects/{project_id}/runs/{run_id}/approval
```

**Request:**
```json
{
  "decision_id": "uuid",
  "action": "approve"
}
```

## Evidence Endpoints

### Get Evidence
```
GET /projects/{project_id}/evidence/{evidence_id}
```

### List Evidence
```
GET /projects/{project_id}/evidence?source_type=tool_output&related_finding_id={id}&page=1&page_size=20
```

## Model Leaderboard Endpoints

### Get Model Leaderboard
```
GET /models/leaderboard?task_type=research_planning
```

**Response:**
```json
{
  "success": true,
  "data": [
    {"model": "gpt-4", "provider": "openai", "success_rate": 0.94, "avg_cost_usd": 1.84, "avg_latency_s": 42, "runs": 50},
    {"model": "claude-3-opus", "provider": "anthropic", "success_rate": 0.91, "avg_cost_usd": 0.92, "avg_latency_s": 31, "runs": 45},
    {"model": "qwen-local", "provider": "ollama", "success_rate": 0.84, "avg_cost_usd": 0.18, "avg_latency_s": 19, "runs": 30}
  ]
}
```

## Health & System Endpoints

### Health Check
```
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "database": "connected",
  "timestamp": "datetime"
}
```

### Get Configuration
```
GET /config
```

**Response:**
```json
{
  "llm_providers": ["openai", "anthropic", "gemini", "ollama", "vllm", "mlx"],
  "governance_frameworks": ["nist_ai_rmf", "iso_42001", "enterprise_ai"],
  "redteam_scenarios": [...],
  "max_experiments": 10,
  "max_agent_steps": 50,
  "max_cost_usd": 5.0,
  "sandbox_enabled": true,
  "network_access": false
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| NOT_FOUND | 404 | Resource not found |
| UNAUTHORIZED | 401 | Invalid or missing API key |
| FORBIDDEN | 403 | Action not permitted |
| CONFLICT | 409 | Resource conflict |
| INTERNAL_ERROR | 500 | Server error |
| AGENT_RUNNING | 409 | Agent already running for this resource |
| APPROVAL_REQUIRED | 403 | Human approval required |
| APPROVAL_REJECTED | 403 | Human rejected approval |
| BUDGET_EXCEEDED | 400 | Cost budget exceeded |
| SANDBOX_ERROR | 500 | Sandbox execution failed |
| TOOL_ERROR | 500 | Tool execution failed |