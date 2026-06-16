# claw-collections

Reusable **OpenClaw workspace bundles** ("collections") — agent files, skills,
cron jobs, plugins, and exec approvals that seed a Claw's filesystem at startup.

A collection is a directory tree. The [claw-operator](https://github.com/redhat-et/claw-operator)
seeds it into a running Claw's persistent volume on first boot, via
`spec.agentFiles` on the `Claw` resource. You point a Claw at a collection in
one of two ways:

- **Git** — `spec.agentFiles.git` (URL + optional `ref`/`path`), or
- **Upload** — the [deployer dashboard](https://github.com/redhat-et/claw-operator-dashboard)
  packages a folder and wires it up for you.

> Seeding works with both **operator-managed** (the default) and **user-managed**
> (`spec.config.management: user`) modes. It runs **once on first boot** (apply
> policy `IfMissing`); edits made later inside the running Claw are preserved
> across restarts. In operator-managed mode, the operator adds infrastructure
> skills (PLATFORM.md, KUBERNETES.md) on top of what `agentFiles` seeds.

## Examples

| Collection | What it shows |
|---|---|
| [`software-qa-mcp`](software-qa-mcp/) | Software Q&A agent backed by an MCP server (Context7), plus a verification sub-agent and a per-workspace skill |
| [`enterprise-profiles`](enterprise-profiles/) | Per-department assistant profiles (HR, Finance, Executive) for the enterprise onboarding pattern |

_New collections go here — one row per top-level directory._

### Selecting a collection over Git

This repo holds many collections, one per top-level directory. Point a Claw at
one with:

| Field (`spec.agentFiles.git` / deployer) | Value |
|---|---|
| `url` / Git URL | `https://github.com/redhat-et/claw-collections.git` |
| `ref` / Git ref | a **branch or tag** (e.g. `main`) — **not** a commit SHA |
| `path` / Path in repo | the bundle directory, e.g. `my-bundle` |

The `path` is the directory that directly contains `workspace-main/`,
`openclaw.json`, etc. — its contents become the bundle root. Leave `path` empty
only if the bundle *is* the whole repository root.

> `ref` must be a branch or tag: the operator clones with `git clone --branch`,
> which does not accept commit SHAs.

---

## How a collection maps onto disk

The Claw mounts its OpenClaw home at `~/.openclaw` (on its PVC). Each
**top-level entry** of your collection is placed there like this:

| In your collection | Lands at | Notes |
|---|---|---|
| `openclaw.json` | merged into the Claw's effective config | **Not** copied as a file — it's the config seed |
| `workspace-main/` | `~/.openclaw/workspace/` | The main agent's workspace |
| `workspace-<id>/` | `~/.openclaw/workspace-<id>/` | One per sub-agent (see below) |
| `skills/` | `~/.openclaw/skills/` | Shared skills (requires config — see below) |
| `cron/jobs.json` | `~/.openclaw/cron/jobs.json` | Scheduled jobs |
| `plugins/` | `~/.openclaw/plugins/` | Requires config — see below |
| `exec-approvals.json` | `~/.openclaw/exec-approvals.json` | Pre-approved exec allowlist |

**A collection is more than files.** Skills directories, plugin paths, cron
enablement, and sub-agent workspaces all have to be declared in your
`openclaw.json` — OpenClaw will not pick them up by location alone. The rest of
this doc is the wiring you need.

---

## `openclaw.json` — the config seed

Your `openclaw.json` is deep-merged into the Claw's effective config. This is
where agents, skills discovery, plugin paths, and cron live.

```jsonc
{
  "agents": {
    // Do NOT set agents.defaults.workspace here — leave it unset so each
    // agent resolves to the workspace path you give it below.
    "list": [
      {
        "id": "default",
        "default": true,
        "name": "Main",
        "workspace": "~/.openclaw/workspace",
        "subagents": { "allowAgents": ["builder", "research"] }
      },
      { "id": "builder",  "name": "Builder",  "workspace": "~/.openclaw/workspace-builder" },
      { "id": "research", "name": "Research", "workspace": "~/.openclaw/workspace-research" }
    ]
  },

  // Needed ONLY for a shared top-level skills/ dir (see Skills below).
  "skills":  { "load": { "extraDirs": ["~/.openclaw/skills"] } },

  // Needed ONLY if you ship a plugins/ dir (see Plugins below).
  "plugins": { "load": { "paths": ["~/.openclaw/plugins"] } },

  // Needed to run cron/jobs.json.
  "cron":    { "enabled": true }
}
```

> Provider/model credentials are managed by the Claw `spec` (the deployer fills
> these in) — don't put API keys in `openclaw.json`.

---

## The main agent — `workspace-main/`

Seeds `~/.openclaw/workspace/`, the default agent's working directory. Typical
contents (all optional):

```
workspace-main/
├── AGENTS.md        # operating instructions for the agent
├── SOUL.md          # persona / style / principles
├── IDENTITY.md      # name, vibe
├── USER.md          # who the human is
├── MEMORY.md        # root memory file (auto-indexed)
├── memory/          # additional memory files (*.md, auto-indexed)
└── skills/          # per-agent skills (auto-discovered, see below)
    └── <skill-name>/SKILL.md
```

## Sub-agents — `workspace-<id>/`

One sibling directory per sub-agent, where `<id>` matches an entry in
`agents.list`. Each lands at `~/.openclaw/workspace-<id>/`.

**Important:** OpenClaw resolves a sub-agent's workspace from its `workspace`
field. Because collections are seeded as siblings, give every sub-agent an
explicit `"workspace": "~/.openclaw/workspace-<id>"` in `openclaw.json` (and do
*not* set `agents.defaults.workspace`, or sub-agents resolve to
`~/.openclaw/workspace/<id>` instead and won't find their seeded files).

```
workspace-builder/
├── AGENTS.md
└── SOUL.md
```

## Skills

Skills are discovered **per-workspace** by default, at
`<workspace>/skills/<name>/SKILL.md`. So the simplest place for a skill is
inside the agent that uses it:

```
workspace-main/skills/docs-answering/SKILL.md      # auto-discovered, no config
```

For a **shared** skills library used across agents, put it at the top level and
register the directory in `openclaw.json`:

```
skills/docs-answering/SKILL.md                      # -> ~/.openclaw/skills/
```
```jsonc
{ "skills": { "load": { "extraDirs": ["~/.openclaw/skills"] } } }
```
A top-level `skills/` is **not** discovered without that `extraDirs` entry.

## Cron jobs — `cron/jobs.json`

Lands at `~/.openclaw/cron/jobs.json` (OpenClaw's default cron store). Enable
cron in config:

```jsonc
{ "cron": { "enabled": true } }
```
```jsonc
// cron/jobs.json
{ "version": 1, "jobs": [ /* job entries per OpenClaw's cron schema */ ] }
```

## Plugins — `plugins/`

There is **no** default `~/.openclaw/plugins` scan. Ship the dir and point the
loader at it:

```
plugins/<plugin>/...                                # -> ~/.openclaw/plugins/
```
```jsonc
{ "plugins": { "load": { "paths": ["~/.openclaw/plugins"] } } }
```

## Exec approvals — `exec-approvals.json`

Lands at `~/.openclaw/exec-approvals.json` and seeds the command allowlist on
boot. Use it to pre-approve commands the agent may run non-interactively. Follow
OpenClaw's exec-approvals format.

## MCP servers — NOT a bundle file

OpenClaw does **not** read a loose `~/.openclaw/mcp.json`. Declare MCP servers on
the **Claw resource** (`spec.mcpServers`) instead of in the collection. (If you
have an old installer-style `mcp.json`, migrate it to the Claw spec.)

---

## Example collection

```
my-collection/
├── openclaw.json                 # agents + skills/plugins/cron wiring
├── workspace-main/
│   ├── AGENTS.md
│   ├── SOUL.md
│   └── skills/
│       └── triage/SKILL.md       # per-agent skill (auto-discovered)
├── workspace-builder/            # sub-agent (needs explicit workspace in openclaw.json)
│   └── AGENTS.md
├── skills/
│   └── shared-research/SKILL.md  # shared skill (needs skills.load.extraDirs)
├── cron/
│   └── jobs.json
├── plugins/
│   └── my-plugin/...
└── exec-approvals.json
```

---

## Quick checklist

- [ ] Claw CR with `spec.agentFiles` pointing at your collection
- [ ] Main agent files under `workspace-main/`
- [ ] Each sub-agent has a `workspace-<id>/` dir **and** an explicit `workspace`
      in `openclaw.json`; `agents.defaults.workspace` is left unset
- [ ] Per-agent skills under `workspace-*/skills/`; shared skills under
      `skills/` **with** `skills.load.extraDirs`
- [ ] `cron.enabled: true` if you ship `cron/jobs.json`
- [ ] `plugins.load.paths: ["~/.openclaw/plugins"]` if you ship `plugins/`
- [ ] MCP servers go on the Claw `spec.mcpServers`, not in the collection
- [ ] No API keys in `openclaw.json` (credentials are managed on the Claw spec)
