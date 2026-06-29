# CI/CD Workflow Example

| Field | Value |
|-------|-------|
| Doc ID | `dcp-example-02` |
| Category | Examples |
| Status | draft |
| Version | 0.1.0-draft |
| Depends on | dcp-core-09, dcp-api-02 |

---

## Summary

GitHub Actions workflow gating production domain changes: compile diff → simulate → test → submit with approved plan hash.

---

## Repository Layout

```
domain/
  production.intent.yaml    # source of truth
  staging.intent.yaml
  tests/
    assertions.yaml
.github/
  workflows/
    domain-deploy.yml
```

---

## Intent File (`domain/production.intent.yaml`)

```yaml
apiVersion: dcp.dev/v1
kind: DomainIntent
metadata:
  domain: example.com
  environment: production
spec:
  routes:
    - host: api.example.com
      traffic:
        origin: https://api-prod.k8s.example.internal
      tls:
        mode: auto
    - host: staging.api.example.com
      traffic:
        origin: https://api-stg.k8s.example.internal
      tls:
        mode: auto
  tests:
    - name: api-prod-health
      host: api.example.com
      assert:
        http:
          url: https://api.example.com/health
          status: 200
```

---

## GitHub Actions Workflow

```yaml
name: Domain Deploy

on:
  push:
    branches: [main]
    paths: ['domain/**']
  pull_request:
    paths: ['domain/**']

env:
  DCP_API: https://api.dcp.dev/v1
  DCP_DOMAIN: example.com

jobs:
  domain-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install DCP CLI
        run: curl -fsSL https://cli.dcp.dev/install.sh | sh

      - name: Compile diff
        id: compile
        env:
          DCP_TOKEN: ${{ secrets.DCP_CI_TOKEN }}
        run: |
          dcp compile diff \
            --domain $DCP_DOMAIN \
            --intent domain/production.intent.yaml \
            --base remote \
            --output plan.json
          echo "plan_hash=$(jq -r .plan_hash plan.json)" >> $GITHUB_OUTPUT

      - name: Simulate
        run: |
          dcp simulate --plan plan.json --output sim.json
          jq -e '.risks_introduced | length == 0' sim.json

      - name: Run domain tests (dry)
        run: dcp test --intent domain/production.intent.yaml --offline

      - name: Policy evaluate
        run: |
          dcp policy evaluate --plan plan.json

      - name: Comment plan on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const plan = require('./plan.json');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              body: `## Domain Plan\n\`\`\`json\n${JSON.stringify(plan, null, 2)}\n\`\`\`\nPlan hash: \`${{ steps.compile.outputs.plan_hash }}\``
            });

  domain-deploy:
    needs: domain-gate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Submit transaction
        env:
          DCP_TOKEN: ${{ secrets.DCP_CI_TOKEN }}
          PLAN_HASH: ${{ needs.domain-gate.outputs.plan_hash }}
        run: |
          dcp transactions submit \
            --domain $DCP_DOMAIN \
            --intent domain/production.intent.yaml \
            --idempotency-key "${{ github.sha }}" \
            --approved-plan-hash "$PLAN_HASH" \
            --wait-for routing,committed

      - name: Verify post-deploy tests
        run: dcp test --intent domain/production.intent.yaml --wait 300
```

---

## Capability Token for CI

Issue via Domain OAuth client credentials:

```json
{
  "name": "github-actions-prod",
  "capabilities": [
    { "action": "txn:submit", "resource": "environment:production" },
    { "action": "route:write", "resource": "fqdn:*.example.com" },
    { "action": "intent:read", "resource": "zone:example.com" }
  ],
  "constraints": {
    "require_approved_plan_hash": true
  }
}
```

---

## Rollback Workflow (Manual Dispatch)

```yaml
name: Domain Rollback
on: workflow_dispatch
  inputs:
    transaction_id:
      required: true
jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - run: |
          dcp transactions rollback \
            --id ${{ inputs.transaction_id }} \
            --wait-for routing
```

---

## Expected Outcomes

| Step | Pass criteria |
|------|---------------|
| Compile diff | Plan generated; no `COMPILE_CONFLICT` |
| Simulate | Zero new takeover risks |
| Test | All assertions pass offline |
| Submit | `routing: active` < 30s |
| Post test | Live probes pass within 5 min |