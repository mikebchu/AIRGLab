# CLI Contracts: AI Research & Governance Platform

**Feature**: AI Research & Governance Platform
**Date**: 2026-09-12
**Status**: Complete

## Overview

The platform provides a `lab` CLI command for all core operations. Built with Typer for type-safe CLI definitions.

## Global Options

```
lab [OPTIONS] COMMAND [ARGS]...

Options:
  --config FILE       Configuration file path (default: ./lab.yaml)
  --project DIR       Project directory (default: current directory)
  --api-url URL       API server URL (default: http://localhost:8000)
  --api-key KEY       API key for authentication
  --verbose, -v       Verbose output
  --quiet, -q         Quiet output (errors only)
  --json              Output as JSON
  --help              Show help
  --version           Show version
```

## Commands

### Project Management

#### Initialize Project
```
lab init [OPTIONS] [NAME]

Options:
  --description TEXT    Project description
  --repository URL      Git repository URL
  --force               Overwrite existing project

Example:
  lab init my-research --description "Fraud detection research"
```

#### Show Project Info
```
lab project show [OPTIONS]

Options:
  --json                Output as JSON
```

### Research Workflow

#### Run Research
```
lab research [OPTIONS]

Options:
  --question TEXT           Research question (required)
  --objective TEXT          Specific objective (required)
  --constraints JSON        Constraints as JSON (e.g., '{"precision": ">=0.95"}')
  --success-criteria JSON   Success criteria as JSON
  --dataset PATH            Dataset file path (required)
  --max-experiments INT     Maximum experiments (default: 10)
  --budget FLOAT            Budget in USD (default: 5.0)
  --model TEXT              Model to use (optional, uses default)
  --wait / --no-wait        Wait for completion (default: --wait)
  --approval-mode MODE      Approval mode: auto, prompt, never (default: prompt)

Example:
  lab research \
    --question "Improve fraud recall while maintaining precision >= 95%" \
    --objective "Maximize recall subject to precision constraint" \
    --constraints '{"precision": ">=0.95"}' \
    --dataset ./data/fraud.csv \
    --max-experiments 10 \
    --budget 5.0
```

#### Show Research Status
```
lab research status [OPTIONS] RESEARCH_ID

Options:
  --watch, -w             Watch for updates
  --json                  Output as JSON
```

#### List Research Questions
```
lab research list [OPTIONS]

Options:
  --status STATUS         Filter by status
  --json                  Output as JSON
```

#### Show Research Report
```
lab research report [OPTIONS] RESEARCH_ID

Options:
  --format FORMAT         Output format: text, json, markdown (default: text)
  --output FILE           Save to file
  --json                  Output as JSON
```

### Experiment Management

#### List Experiments
```
lab experiments list [OPTIONS]

Options:
  --research-id UUID      Filter by research question
  --status STATUS         Filter by status
  --json                  Output as JSON
```

#### Show Experiment Details
```
lab experiments show [OPTIONS] EXPERIMENT_ID

Options:
  --json                  Output as JSON
```

#### Compare Experiments
```
lab experiments compare [OPTIONS] EXPERIMENT_ID_1 EXPERIMENT_ID_2

Options:
  --metrics TEXT          Comma-separated metrics to compare
  --json                  Output as JSON
```

### Governance Workflow

#### Run Governance Assessment
```
lab govern [OPTIONS]

Options:
  --system-name TEXT          System name (required)
  --system-file PATH          System description file (YAML/JSON) (required)
  --framework TEXT            Framework: nist_ai_rmf, iso_42001, enterprise_ai (default: nist_ai_rmf)
  --framework-version TEXT    Framework version (default: latest)
  --wait / --no-wait          Wait for completion (default: --wait)

Example:
  lab govern \
    --system-name "Cosmetics Recommendation Assistant" \
    --system-file ./examples/recommendation_system/system.yaml \
    --framework nist_ai_rmf
```

#### Show Governance Assessment
```
lab govern show [OPTIONS] ASSESSMENT_ID

Options:
  --json                  Output as JSON
```

#### List Governance Assessments
```
lab govern list [OPTIONS]

Options:
  --status STATUS         Filter by status
  --json                  Output as JSON
```

#### Show Governance Report
```
lab govern report [OPTIONS] ASSESSMENT_ID

Options:
  --format FORMAT         Output format: text, json, markdown (default: text)
  --output FILE           Save to file
  --json                  Output as JSON
```

### Red Team Workflow

#### Run Red Team Assessment
```
lab redteam [OPTIONS]

Options:
  --target-name TEXT        Target system name (required)
  --target-file PATH        Target description file (YAML/JSON) (required)
  --target-type TYPE        Target type: llm_app, agent, api, prompt, tool, data_flow (required)
  --scenarios TEXT          Comma-separated scenarios (default: all)
  --wait / --no-wait        Wait for completion (default: --wait)

Example:
  lab redteam \
    --target-name "Vulnerable Chatbot" \
    --target-file ./examples/llm_chatbot/target.yaml \
    --target-type llm_app \
    --scenarios "prompt_injection,tool_abuse,secret_exposure"
```

#### Show Red Team Results
```
lab redteam show [OPTIONS] RUN_ID

Options:
  --json                  Output as JSON
```

#### List Red Team Runs
```
lab redteam list [OPTIONS]

Options:
  --status STATUS         Filter by status
  --json                  Output as JSON
```

#### Show Red Team Report
```
lab redteam report [OPTIONS] RUN_ID

Options:
  --format FORMAT         Output format: text, json, markdown (default: text)
  --output FILE           Save to file
  --json                  Output as JSON
```

### Agent Run Management

#### List Agent Runs
```
lab run list [OPTIONS]

Options:
  --agent-type TYPE       Filter by agent type
  --status STATUS         Filter by status
  --project-id UUID       Filter by project
  --json                  Output as JSON
```

#### Show Agent Run Details
```
lab run show [OPTIONS] RUN_ID

Options:
  --json                  Output as JSON
```

#### Show Agent Trace
```
lab run trace [OPTIONS] RUN_ID

Options:
  --follow, -f            Follow live trace (WebSocket)
  --event-type TYPE       Filter by event type
  --json                  Output as JSON
```

#### Approve/Reject Pending Action
```
lab run approve [OPTIONS] RUN_ID DECISION_ID

Options:
  --action ACTION         approve or reject (required)
  --reason TEXT           Reason for decision
```

### Evidence Management

#### List Evidence
```
lab evidence list [OPTIONS]

Options:
  --source-type TYPE      Filter by source type
  --finding-id UUID       Filter by finding
  --experiment-id UUID    Filter by experiment
  --json                  Output as JSON
```

#### Show Evidence Details
```
lab evidence show [OPTIONS] EVIDENCE_ID

Options:
  --json                  Output as JSON
```

### Model Leaderboard

#### Show Model Leaderboard
```
lab models leaderboard [OPTIONS]

Options:
  --task-type TYPE        Filter by task type
  --json                  Output as JSON
```

### Testing

#### Run Tests
```
lab test [OPTIONS]

Options:
  --unit                  Run unit tests only
  --integration           Run integration tests only
  --agent                 Run agent tests only
  --security              Run security tests only
  --coverage              Generate coverage report
  --verbose, -v           Verbose output
```

### Configuration

#### Show Configuration
```
lab config show [OPTIONS]

Options:
  --json                  Output as JSON
```

#### Set Configuration
```
lab config set [OPTIONS] KEY VALUE

Options:
  --global                Set globally (user config)
```

#### Validate Configuration
```
lab config validate [OPTIONS]
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Authentication failed |
| 4 | Resource not found |
| 5 | API error |
| 6 | Approval rejected |
| 7 | Budget exceeded |
| 8 | Sandbox error |
| 9 | Tool execution error |
| 10 | Validation error |

## Configuration File (lab.yaml)

```yaml
api:
  url: "http://localhost:8000"
  key: "your-api-key"

project:
  default: "./projects/default"

llm:
  provider: "openai"
  model: "gpt-4"
  base_url: null
  api_key: null

execution:
  max_experiments: 10
  max_agent_steps: 50
  max_cost_usd: 5.0
  sandbox_enabled: true
  network_access: false

approval:
  mode: "prompt"  # auto, prompt, never
  high_impact_actions:
    - "production_deployment"
    - "destructive_filesystem"
    - "external_pr"
    - "network_access"
    - "sensitive_data"
    - "budget_exceeded"
    - "security_policy_change"

security:
  command_guard_enabled: true
  secret_redaction_enabled: true
  prompt_injection_detection: true

observability:
  tracing_enabled: true
  metrics_enabled: true
  langfuse_enabled: false
```

## Environment Variables

All configuration options can be set via environment variables with prefix `LAB_`:

```
LAB_API_URL=http://localhost:8000
LAB_API_KEY=your-key
LAB_LLM_PROVIDER=openai
LAB_LLM_MODEL=gpt-4
LAB_MAX_EXPERIMENTS=10
LAB_MAX_COST_USD=5.0
LAB_SANDBOX_ENABLED=true
LAB_NETWORK_ACCESS=false
LAB_APPROVAL_MODE=prompt
```