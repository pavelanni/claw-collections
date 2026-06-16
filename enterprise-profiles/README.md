# Enterprise Department Profiles

Pre-built workspace profiles for common enterprise roles. Each
profile includes persona files (SOUL.md, AGENTS.md, TOOLS.md),
identity, and role-specific skills. Designed for the
per-department deployment pattern where ITOps seeds tailored
assistants from a Git repo.

## Profiles

| Profile | Role | Skill |
|---|---|---|
| `hr-specialist` | HR policy, communications, onboarding | Onboarding checklist generator |
| `financial-analyst` | Financial analysis, modeling, reporting | Budget variance analysis |
| `executive-assistant` | Meeting prep, exec comms, strategic docs | Executive brief summarizer |

## Deployment

These profiles work with both operator-managed and user-managed
modes. Point a Claw CR at a profile directory:

### HR Specialist

```yaml
apiVersion: claw.sandbox.redhat.com/v1alpha1
kind: Claw
metadata:
  name: hr-assistant
  namespace: ai-assistants
spec:
  agentFiles:
    git:
      url: https://github.com/redhat-et/claw-collections.git
      ref: main
      path: enterprise-profiles/hr-specialist
  credentials:
    - name: anthropic
      provider: anthropic
      secretRef:
        - name: shared-anthropic
          key: api-key
```

### Financial Analyst

```yaml
apiVersion: claw.sandbox.redhat.com/v1alpha1
kind: Claw
metadata:
  name: finance-assistant
  namespace: ai-finance
spec:
  agentFiles:
    git:
      url: https://github.com/redhat-et/claw-collections.git
      ref: main
      path: enterprise-profiles/financial-analyst
  credentials:
    - name: anthropic
      provider: anthropic
      secretRef:
        - name: shared-anthropic
          key: api-key
```

### Executive Assistant

```yaml
apiVersion: claw.sandbox.redhat.com/v1alpha1
kind: Claw
metadata:
  name: exec-assistant
  namespace: ai-executive
spec:
  agentFiles:
    git:
      url: https://github.com/redhat-et/claw-collections.git
      ref: main
      path: enterprise-profiles/executive-assistant
  credentials:
    - name: anthropic
      provider: anthropic
      secretRef:
        - name: shared-anthropic
          key: api-key
```

## What happens on pod start

1. init-config clones the profile directory from this repo
2. `workspace-main/` is seeded into the workspace (SOUL.md,
   AGENTS.md, TOOLS.md, skills)
3. `openclaw.json` provides the agent name and model preference
4. In operator-managed mode (the default), the operator adds
   infrastructure skills (PLATFORM.md, KUBERNETES.md) on top
5. The operator's default AGENTS.md and SOUL.md use
   `seedIfMissing` — they do not overwrite the profile's
   versions

## Customization

These profiles are starting points. Common customizations:

- **Add `applyPolicy: Always`** to re-seed on every restart
  (for controlled environments where drift is unacceptable)
- **Add `restrictions.personaRef`** to lock SOUL.md read-only
  (prevents the agent from modifying its own constraints)
- **Change the model** in `openclaw.json` to match your
  available providers
- **Add MCP servers** on the Claw CR for domain-specific data
  access (e.g., an HR data API, financial data service)
