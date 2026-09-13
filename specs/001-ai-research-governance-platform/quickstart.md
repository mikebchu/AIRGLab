# Quickstart Validation Guide: AI Research & Governance Platform

**Feature**: AI Research & Governance Platform
**Date**: 2026-09-12
**Status**: Complete

## Prerequisites

- Docker and Docker Compose installed
- Python 3.12+ (for local development)
- Node.js 18+ (for frontend development)
- At least one LLM provider configured:
  - OpenAI API key, OR
  - Anthropic API key, OR
  - Local model endpoint (Ollama, vLLM, MLX)

## Quick Start (Docker)

### 1. Clone and Configure

```bash
git clone <repository-url>
cd ai-research-governance-lab
cp .env.example .env
# Edit .env with your LLM provider credentials
```

### 2. Start Platform

```bash
docker-compose up -d
```

### 3. Verify Health

```bash
curl http://localhost:8000/api/v1/health
# Expected: {"status": "healthy", "version": "1.0.0", "database": "connected"}
```

### 4. Access Web UI

Open http://localhost:3000 in browser.

### 5. Initialize Project via CLI

```bash
# Install CLI (if not using Docker)
pip install -e ./apps/cli

# Or use Docker
docker-compose exec api lab init demo-project --description "Demo project for validation"
```

## Validation Scenarios

### Scenario 1: Fraud Detection Research Demo (Research Mode)

**Objective**: Verify autonomous ML research workflow with evidence-backed results.

**Prerequisites**: Platform running, LLM configured.

**Steps**:

1. **Initialize project** (if not done):
   ```bash
   lab init fraud-research --description "Fraud detection research demo"
   ```

2. **Run research**:
   ```bash
   lab research \
     --question "Improve fraud recall while maintaining precision >= 95%" \
     --objective "Maximize recall subject to precision constraint" \
     --constraints '{"precision": ">=0.95"}' \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 5 \
     --budget 2.0 \
     --wait
   ```

3. **Monitor progress** (in another terminal):
   ```bash
   lab research status <RESEARCH_ID> --watch
   ```

4. **View trace** (real-time):
   ```bash
   lab run trace <LATEST_RUN_ID> --follow
   ```

5. **Verify completion**:
   - Research status shows "completed"
   - At least 3 experiments executed
   - Each experiment has: dataset_version, code_commit, model_version, parameters, seed, metrics, artifacts, duration, cost
   - Failed experiments preserved with evidence

6. **Generate report**:
   ```bash
   lab research report <RESEARCH_ID> --format markdown --output research-report.md
   ```

**Expected Outcomes** (Success Criteria):
- ✅ SC-001: Complete demo in under 10 minutes
- ✅ SC-002: Full research cycle with ≥3 experiments, evidence persisted
- ✅ SC-005: Complete execution trace viewable
- ✅ SC-006: Report distinguishes Observed/Inferred/Recommended/Not Tested/Unknown
- ✅ SC-012: New developer can run without code changes

**Verification Commands**:
```bash
# Check experiments
lab experiments list --research-id <RESEARCH_ID> --json

# Check experiment details
lab experiments show <EXPERIMENT_ID> --json

# Verify evidence
lab evidence list --experiment-id <EXPERIMENT_ID> --json

# Check agent trace
lab run trace <RUN_ID> --json
```

### Scenario 2: Cosmetics Recommendation Governance Demo (Governance Mode)

**Objective**: Verify AI governance assessment against NIST AI RMF with evidence-backed findings.

**Prerequisites**: Platform running, LLM configured.

**Steps**:

1. **Run governance assessment**:
   ```bash
   lab govern \
     --system-name "Cosmetics Recommendation Assistant" \
     --system-file ./examples/recommendation_system/system.yaml \
     --framework nist_ai_rmf \
     --wait
   ```

2. **Monitor progress**:
   ```bash
   lab govern show <ASSESSMENT_ID> --watch
   ```

3. **Generate report**:
   ```bash
   lab govern report <ASSESSMENT_ID> --format markdown --output governance-report.md
   ```

**Expected Outcomes** (Success Criteria):
- ✅ SC-003: Risk-scored report with ≥5 findings linked to controls and evidence
- ✅ SC-005: Execution trace available
- ✅ SC-006: Report distinguishes claim types
- ✅ SC-011: Runs in Docker, passes quality gates

**Verification Commands**:
```bash
# Check findings
lab govern show <ASSESSMENT_ID> --json | jq '.data.findings'

# Verify evidence linkage
lab evidence list --finding-id <FINDING_ID> --json

# Check risk breakdown
lab govern show <ASSESSMENT_ID> --json | jq '.data.risk_breakdown'
```

### Scenario 3: Vulnerable AI Assistant Red Team Demo (Red Team Mode)

**Objective**: Verify safe adversarial testing identifies vulnerabilities with evidence.

**Prerequisites**: Platform running, LLM configured.

**Steps**:

1. **Run red team assessment**:
   ```bash
   lab redteam \
     --target-name "Vulnerable Chatbot" \
     --target-file ./examples/llm_chatbot/target.yaml \
     --target-type llm_app \
     --scenarios "prompt_injection,tool_abuse,secret_exposure,command_injection" \
     --wait
   ```

2. **Monitor progress**:
   ```bash
   lab redteam show <RUN_ID> --watch
   ```

3. **Generate report**:
   ```bash
   lab redteam report <RUN_ID> --format markdown --output redteam-report.md
   ```

**Expected Outcomes** (Success Criteria):
- ✅ SC-004: Identifies ≥3 vulnerabilities (prompt injection, tool auth, secret protection) with evidence
- ✅ No real credentials accessed
- ✅ No external systems impacted
- ✅ SC-005: Execution trace available
- ✅ SC-009: Security controls block dangerous commands, redact secrets

**Verification Commands**:
```bash
# Check test results
lab redteam show <RUN_ID> --json | jq '.data.tests[] | select(.result=="fail")'

# Verify evidence
lab evidence list --redteam-test-id <TEST_ID> --json

# Check security findings
lab redteam show <RUN_ID> --json | jq '.data.summary'
```

### Scenario 4: Human-in-the-Loop Approval

**Objective**: Verify approval workflow for high-impact actions.

**Prerequisites**: Platform running, approval mode set to "prompt".

**Steps**:

1. **Configure approval mode**:
   ```bash
   lab config set approval.mode prompt
   ```

2. **Run research with network access request** (simulated):
   ```bash
   # This would trigger approval request in real scenario
   lab research \
     --question "Test approval workflow" \
     --objective "Verify approval" \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 1
   ```

3. **When approval requested**, check CLI or UI for prompt:
   ```
   APPROVAL REQUIRED
   Agent wants to: Enable network access for model download
   Reason: Download external pretrained model
   Risk: Medium
   [Approve] [Reject]
   ```

4. **Approve or reject** and verify decision recorded.

**Expected Outcomes** (Success Criteria):
- ✅ SC-007: System enforces human approval, records decisions
- ✅ Article VI compliance: Explicit approval, visible action details, recorded decisions

### Scenario 5: Model-Agnostic Execution

**Objective**: Verify agents work with multiple model providers.

**Prerequisites**: At least 2 model providers configured.

**Steps**:

1. **Test with OpenAI**:
   ```bash
   LLM_PROVIDER=openai LLM_MODEL=gpt-4 lab research \
     --question "Quick test" \
     --objective "Test model" \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 1
   ```

2. **Test with Anthropic**:
   ```bash
   LLM_PROVIDER=anthropic LLM_MODEL=claude-3-opus lab research \
     --question "Quick test" \
     --objective "Test model" \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 1
   ```

3. **Test with local model** (if available):
   ```bash
   LLM_BASE_URL=http://host.docker.internal:11434/v1 LLM_MODEL=qwen2.5 lab research \
     --question "Quick test" \
     --objective "Test model" \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 1
   ```

**Expected Outcomes** (Success Criteria):
- ✅ SC-008: Agents execute with ≥3 providers without code changes
- ✅ Article VII compliance: Model abstraction works

### Scenario 6: Security Controls Verification

**Objective**: Verify security-by-default controls.

**Prerequisites**: Platform running.

**Steps**:

1. **Test command guard**:
   ```bash
   # This should be blocked
   docker-compose exec api python -c "
   from tools.shell import ShellTool
   tool = ShellTool()
   result = tool.execute('rm -rf /')
   print(result)
   "
   ```

2. **Test secret redaction**:
   ```bash
   docker-compose exec api python -c "
   from security.secrets import redact_secrets
   text = 'API_KEY=sk-abc123 password=secret123'
   print(redact_secrets(text))
   "
   # Expected: API_KEY=*** password=***
   ```

3. **Test network isolation**:
   ```bash
   docker-compose exec api python -c "
   from tools.python import PythonTool
   tool = PythonTool()
   result = tool.execute('import requests; requests.get(\"http://httpbin.org/get\")')
   print(result)
   "
   # Expected: Network error (network=none)
   ```

**Expected Outcomes** (Success Criteria):
- ✅ SC-009: Dangerous commands blocked, secrets redacted, network disabled
- ✅ Article V compliance: Security by default

### Scenario 7: Cost Tracking Verification

**Objective**: Verify cost tracking per agent run and experiment.

**Prerequisites**: Platform running, LLM configured.

**Steps**:

1. **Run research with budget**:
   ```bash
   lab research \
     --question "Cost tracking test" \
     --objective "Verify cost tracking" \
     --dataset ./data/demo/fraud.csv \
     --max-experiments 2 \
     --budget 1.0
   ```

2. **Check cost in report**:
   ```bash
   lab research report <RESEARCH_ID> --json | jq '.data.reproducibility.cost_usd'
   ```

3. **Check agent run costs**:
   ```bash
   lab run list --json | jq '.data[] | {id: .id, cost: .estimated_cost_usd, tokens: .total_tokens}'
   ```

4. **Check dashboard** (web UI): Model Leaderboard shows cost per model.

**Expected Outcomes** (Success Criteria):
- ✅ SC-010: Cost tracking records estimated cost per run/experiment
- ✅ Article XV compliance: Observability and cost awareness

### Scenario 8: Quality Gates Verification

**Objective**: Verify all quality gates pass.

**Prerequisites**: Platform running, development environment.

**Steps**:

```bash
# Run all tests
lab test --coverage

# Or via Docker
docker-compose exec api pytest --cov=apps --cov=core --cov=agents --cov=tools --cov=research --cov=governance --cov=redteam --cov=evaluation --cov=memory --cov=storage --cov=observability --cov=security --cov=cli

# Check linting
docker-compose exec api ruff check .

# Check type checking
docker-compose exec api mypy .

# Check security tests
docker-compose exec api pytest tests/security/ -v

# Build Docker images
docker-compose build
```

**Expected Outcomes** (Success Criteria):
- ✅ SC-011: All quality gates pass (ruff, mypy, pytest, security tests, Docker build)
- ✅ Article XIV compliance: Testability and engineering quality

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Database connection failed | Ensure PostgreSQL container is healthy: `docker-compose ps` |
| LLM API errors | Verify API keys in `.env`, check provider status |
| Sandbox execution fails | Check Docker daemon running, permissions |
| Frontend not loading | Ensure Vite dev server running: `docker-compose logs web` |
| CLI not found | Install with `pip install -e ./apps/cli` or use `docker-compose exec api lab` |

### Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f web
docker-compose logs -f db
```

### Reset Environment

```bash
docker-compose down -v
docker-compose up -d
```

## Validation Checklist

Run all scenarios and verify:

- [ ] Scenario 1: Fraud Research Demo - PASS/FAIL
- [ ] Scenario 2: Governance Demo - PASS/FAIL
- [ ] Scenario 3: Red Team Demo - PASS/FAIL
- [ ] Scenario 4: Human Approval - PASS/FAIL
- [ ] Scenario 5: Model Agnostic - PASS/FAIL
- [ ] Scenario 6: Security Controls - PASS/FAIL
- [ ] Scenario 7: Cost Tracking - PASS/FAIL
- [ ] Scenario 8: Quality Gates - PASS/FAIL

**Overall**: All scenarios PASS → MVP validation successful.

## Next Steps

After validation:
1. Review generated reports for evidence quality
2. Check agent traces for completeness
3. Verify constitutional compliance in implementation
4. Proceed to `/speckit-tasks` for implementation task generation