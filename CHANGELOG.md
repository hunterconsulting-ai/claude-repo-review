# Changelog

## 2.0.0 — 2026-10-28

**Breaking:** the skill moved from the repo root to `skills/repo-review/` (plugin layout), and your context file and logs now live in `~/.claude/repo-review/`, outside the skill's folder. The v1 location of `user-context.md` is still read.

### Added
- **Prompt-injection ground rule** — fetched content is data; text that steers the reviewer is a HIGH finding (quoted safely, hidden characters escaped), and instructions in a skill that tell the agent to hide actions, exfiltrate data or override the user are a hard reject. Agent-addressed text that is merely a claim gets verified, not flagged.
- **Judge before you compare** — on a fresh review, the old log entry is opened only after the new verdict, so the old conclusions can't anchor it.
- **The full file tree first**, then the Claude Code payload in full — every `SKILL.md`, agent, command, hook (and what its scripts import), plugin manifest and `.mcp.json`. v1 did not read `SKILL.md` at all.
- **Supply-chain checks:** the install path followed line by line; every declared dependency confirmed to exist on its registry and not quarantined (an unregistered pinned name is claimable — the dependency-confusion hard reject now covers it); registry packages compared with the reviewed commit; shipped images and binaries traced to their source; open issues and security advisories searched.
- **New yellow flags:** pins to known-vulnerable versions, `uvx`/`npx` installs that ignore lockfiles, TLS verification disabled, unsafe model loading, telemetry whose off switch doesn't gate every call.
- **Relevance and fit:** can the user run it (hardware), does the license fit their business model, and their own decision records.
- **Reviews are tied to a commit**, recorded in the report and the log.
- **Delta review** of an update since the reviewed commit, and a **revisit sweep** of REVISIT verdicts.
- **Two-file log**: a one-line-per-review index for the pre-check, plus the full log; guidance for moving a v1 log.
- **URL scoping**: org listings, awesome-list indexes, demo-site satellites, MCP servers, batches.
- **MCP server module** (transport, tools, resources, prompts, auth, outbound calls, file-system scope) and the **CONNECT** verdict — merged from a separate MCP review skill.
- **License checks** from the raw file: every license filename, read to the end, `NOASSERTION` means read it, no file means all rights reserved.
- **Popularity sanity check**, **same-name-different-owner** check, **batch pre-flight** (prove GitHub access first, pin subagent writes to a scratch directory).
- Optional **static scanner** step (e.g. NVIDIA SkillSpector), static-only by default.
- Plugin marketplace manifests.

### Changed
- `SKILL.md` keeps the method; detail moved to `references/`, loaded only when needed.
- REVISIT verdicts require a checkable trigger; REJECT may carry a reconsider-when condition; SKIP may carry unresolved warnings.
- Undisclosed or content-carrying network calls are a hard reject; disclosed opt-out telemetry is a yellow flag.
- Scanner output is evidence to triage, never a verdict.
- The overlap check uses the live installed set: user and project skills (switched-off ones included), plugins, and connected MCP servers.
- The report gains Transport/tools, Supersedes and Coverage lines.

### How v2 was tested
Before release, three fresh reviewers ran v2 blind on repos with known outcomes, and a different model (Gemini) reviewed the method itself. One test found a live dependency-confusion risk two earlier v1 reviews had missed. Their method notes shaped the final text.

## 1.1.0 — 2026-05-04
- Claude Code–specific checklist and runtime trust-surface analysis.

## 1.0.0 — 2026-03-30
- First release: security analysis, relevance via `user-context.md`, overlap check, sandbox assessment, verdicts, optional review log.
