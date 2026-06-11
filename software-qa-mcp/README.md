# software-qa-mcp

A starter collection: a software Q&A agent that answers from **live
documentation** via an MCP server (Context7), with a verification sub-agent.

## What's in it

```
software-qa-mcp/
├── openclaw.json                       # agents + MCP servers (the config seed)
├── exec-approvals.json                 # baseline exec approval policy
├── workspace-main/                     # main agent → ~/.openclaw/workspace/
│   ├── AGENTS.md  SOUL.md  TOOLS.md
│   └── skills/docs-answering/SKILL.md  # per-agent skill (auto-discovered)
└── workspace-docs-checker/             # sub-agent → ~/.openclaw/workspace-docs-checker/
    ├── AGENTS.md  SOUL.md
```

- **MCP** lives in `openclaw.json` under `mcp.servers` (OpenClaw does **not**
  read a loose `mcp.json`). Context7 is a `streamable-http` server.
- The **`docs-checker` sub-agent** declares an explicit
  `workspace: ~/.openclaw/workspace-docs-checker` so its seeded files
  (`workspace-docs-checker/`) are found, and the main agent allows it via
  `agents.list[0].subagents.allowAgents`.
- The **skill** sits in `workspace-main/skills/`, which OpenClaw discovers
  per-workspace automatically (no `skills.load.extraDirs` needed).

## Use it

Point a **user-managed** Claw (`spec.config.management: user`) at this directory
— via the deployer's *OpenClaw workspace source → Git*, or directly on the Claw:

```yaml
spec:
  config:
    management: user
  agentFiles:
    git:
      url: https://github.com/redhat-et/claw-collections.git
      ref: main            # branch or tag (not a commit SHA)
      path: software-qa-mcp
```

Then ask, e.g., "How does the Next.js App Router handle layouts?" — the agent
pulls current docs through Context7 before answering, and can delegate
version-sensitive checks to `docs-checker`.

> Context7's public endpoint may require an API key. If so, add it as a header
> in `openclaw.json`:
> `"context7": { "url": "...", "transport": "streamable-http", "headers": { "Authorization": "Bearer <token>" } }`

See the [top-level README](../README.md) for the full collection layout and the
rules each piece follows.
