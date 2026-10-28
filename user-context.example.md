# User Context — Repo Review

Copy this file to `~/.claude/repo-review/user-context.md` and fill it in.
(Windows: `C:\Users\<you>\.claude\repo-review\user-context.md`.)

Keep it **outside** the skill's own folder — reinstalling or updating the skill replaces that
folder, and anything you saved inside it goes with it. Every section is optional; without this
file the review still runs in full, minus the relevance rating.

---

## My Role
<!-- What do you do? e.g. solo consultant, backend engineer, data scientist, operations manager -->


## My Services or Use Cases
<!-- What do you build or deliver? Used to rate relevance (Step 5). e.g.
- Workflow automation (n8n, Make, Zapier)
- Web apps (Node.js / React)
- Data pipelines and reporting
-->


## My Current Tech Stack
<!-- e.g. Python, TypeScript, PostgreSQL, Docker, Claude Code, n8n, VS Code, Windows 11 -->


## My Hardware
<!-- Used to say whether you can actually run it. e.g. "Windows 11 laptop, 8 GB GPU, 32 GB RAM" -->


## Decision Records
<!-- Optional: files where you record tooling decisions, so a review can respect them.
     e.g. "~/notes/tooling-decisions.md — we read Markdown vaults directly, no MCP bridge" -->


## Security Concerns Specific to My Environment
<!-- Anything beyond the standard checks. e.g.
- I handle client data under NDA — flag any telemetry or data egress
- I work with HIPAA-regulated data — flag any health-data handling
- My working directory is a git repo — subagents must never write into it
-->


## Review Log
<!-- Where reviews are recorded (see references/review-log.md). Defaults if left blank:
     index ~/.claude/repo-review/review-index.md, log ~/.claude/repo-review/review-log.md -->
- Index:
- Log:
- Archive logs (searched in the pre-check, never written to):


## Install Defaults
<!-- e.g. "Install skills user-scoped unless I say otherwise" -->
