# Changelog

## 2.1.0 — 2026-10-29

### Added
- **Local folders and archives** as a review target: a content hash stands in for the commit (a canonical form, with equivalent Git Bash/Linux, macOS and PowerShell commands), a provenance check against the claimed repo's matching tag for the repo metadata, a recursive listing (hidden files, links, NTFS alternate data streams) for the file tree, and dependency advisories for the issue tracker. Archive entries are listed before extraction, an included `.git/` is read as text and never run, and the log key is the folder name — a claimed `owner/name` stays a claim in the report.
- **Plugin marketplaces:** each entry's `source` is followed to the code that actually installs; a `command` source is treated as code that runs at install and every session.
- **The plugin payload** now includes every part that runs or installs code: `bin/`, monitors, LSP servers, MCP bundles, workflows, plugin `dependencies`, npm packages installed at cache time, declared component paths, and scripts a skill tells Claude to run.
- **Ground rule 5 — never print a credential:** config files are read by extracting the key needed, never whole; a hardcoded credential is reported by location and shape, never its value; comparisons show key names, not values, for config and credential files.
- **Remote MCP endpoints** get defined log values (Repo = the endpoint host, Commit = `n/a (hosted)`), and a user-scope route for `${VAR}` credentials (`claude mcp add-json --scope user`).
- **MCP report block** for the trust surface (tools, resources, prompts, authentication, outbound calls, file system), and the overlap criteria — more capable, better maintained, meaningfully different.
- **New yellow flag:** session-lifecycle hooks, which run without any tool call.
- **v1 log traps:** in-body resolution lines, unnumbered reviews, multi-item tables, split verdicts, "reason for REVISIT" vs the trigger, supersession across logs, and `→` as a path separator.
- `user-context.example.md` has slots for your own log formats and numbering.

### Changed
- **Network calls, redrawn:** undisclosed calls, or user content sent somewhere unrelated to the tool's stated function, are a hard reject. **Telemetry** means usage metadata; telemetry that carries user content (prompts, code, file contents or paths, repo names, config) is a hard reject unless it is both opt-in and disclosed before install. A disclosed vendor-hosted service that *is* the product (a remote MCP endpoint, a SaaS wrapper) is a yellow flag weighed in the verdict, as the MCP module already treated it.
- **Reviewer-steering text is a hard reject** as well as a HIGH finding; densely hidden text is quoted as an escaped excerpt plus a labelled decoded rendering.
- **The pre-check no longer reads the old verdict.** It reports only that a prior review exists (id, date, repo, commit); the old verdict is read after the new one, or when you say the old review is enough. A REVISIT sweep that leads to a fresh review runs it in a fresh subagent, and a batch runs Step 1 for every repo before fanning out.
- The overlap check lists MCP servers by name only and reads installed plugins from files, extracting the `enabledPlugins` key, instead of running `claude plugin` subcommands.
- The scanner step names SkillSpector's static-only flag (`--no-llm`), scans a plugin's root and each skill folder, and reports completeness as well as exit status.
- Scratch files always go outside any repo, with or without a context file.
- The MCP module applies to every step, covers apps that embed a server, and takes `--scope` from your context.
- The v1 upgrade path: move your v1 `user-context.md` and `review-log.md` out of the skill folder before updating; when both context files exist, `~/.claude/repo-review/` wins.
- `version` is set in `plugin.json` only, and `marketplace.json` uses the top-level `description`.

### Fixed
- A literal zero-width space in `SKILL.md`'s escape example — it now reads `\u200b`.
- The log entry template no longer repeats the report's header lines.
- Claims that every rule's incident is named in the references now say "where a reference has the case".
- Invisible variation selectors removed, so every file is free of hidden and default-ignorable characters.

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
