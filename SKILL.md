---
name: repo-review
description: "Analyze a GitHub repo for security concerns, relevance to your work, and overlap with existing skills. Trigger when a GitHub URL is provided with a request to review, analyze, or evaluate a repo before installing."
---

# GitHub Repo Security & Relevance Review

## Step 1 — Pre-Check: Review Log (Optional)

Check whether the user has mentioned a review log file in this session.

- **If a log exists:** Search it for the repo URL. If found, show the existing entry (date and verdict) and ask:
  > "This repo was already reviewed on [date] with verdict [verdict]. Would you like a fresh review, or is the existing entry sufficient?"
  Wait for confirmation before proceeding.
- **If no log has been mentioned or found:** Proceed to Step 2. After delivering the final report in Step 8, offer to help the user create one.

## Step 2 — Gather Repo Information

Fetch and read:
- The repo's main README
- Package manifest files (package.json, requirements.txt, pyproject.toml, Cargo.toml, etc.)
- Install scripts (install.sh, setup.py, Makefile targets)
- Any hook definitions or scripts that run automatically
- License file

## Step 3 — Security Analysis

### Hard Rejects (REJECT if any are found)
- Obfuscated or minified code with no readable source
- Hardcoded credentials, API keys, or tokens in source files
- Network calls to unknown or author-controlled endpoints during install or runtime
- Install scripts that pull code from secondary unreviewed sources
- Dependency confusion attack patterns (internal-looking package names on public registries)
- Code that modifies system files outside expected directories

### Yellow Flags (note but do not auto-reject — explain each)
- Unpinned dependencies (`@latest`, `@alpha`, `@beta`, `*`) — supply chain risk
- Optional cloud or telemetry features — note as inactive unless configured
- Hook code that intercepts every Claude Code tool call — assess whether local-only
- Single-author package with no community review or activity
- Proprietary license with unclear or restrictive terms

### Claude Code–Specific Checklist

Claude Code repos carry attack surfaces that generic security reviews miss. For each item below, answer yes / no / unclear and explain any non-no answer:

- Registers hooks (stop, lifecycle, pre/post-tool, or similar)?
- Hook code executes shell commands or spawns external processes?
- Custom commands invoke shell or external tools?
- Writes persistent state files to disk?
- Reads local state files to influence control flow?
- Any execution path runs without explicit user confirmation?
- Side effects of hooks and commands fully documented?
- Failure behavior is safe by default (fails closed, not open)?
- A documented path exists to disable or fully remove the integration?

### Runtime Trust Surface

What this repo can actually do once installed is more important than what it claims to do. Document both, then compare.

**What the repo says it does** (from README, docs, config):
- File system access:
- Network access:
- Hook / execution behavior:
- External APIs or tools:

**What the code actually does** (from static inspection):
- File system access:
- Network access:
- Hook / execution behavior:
- External APIs or tools:

Label each inferred item: confirmed / likely / unclear.

**Gap analysis:** List any behavior present in the code that is absent or understated in the documentation. If none found, state "None identified."

## Step 4 — Relevance Assessment

Check for a `user-context.md` file in the same directory as this skill's `SKILL.md`.

- **If found:** Evaluate the repo against the user's defined role, services, and stack from that file. Rate relevance: **High / Medium / Low / None** with specific reasoning per area.
- **If not found:** Note inline:
  > "No user context configured — relevance assessment skipped. Copy `user-context.example.md` to `user-context.md` in this skill's folder and fill it in to enable this step."
  Proceed without a relevance rating.

## Step 5 — Overlap Check

Compare against skills already installed in `~/.claude/skills/`:
- Does this repo overlap with any currently installed skill?
- If yes: is it more robust, better maintained, or does it cover additional use cases?
- Give a clear recommendation: **keep existing** / **replace** / **install alongside**

If the skills directory cannot be read, note it and skip this step.

## Step 6 — Sandbox Assessment

If the security verdict is inconclusive — not a clear REJECT but not fully verified — state it explicitly:

> "Sandbox testing recommended before local install."

- Preferred method: **GitHub Codespaces** (cloud Linux, nothing runs locally, free 60 hrs/month)
- Provide the exact command(s) to run in the Codespace terminal to test the installer and inspect what gets written

Do not recommend sandboxing for repos that are already clearly clean or clearly rejected.

## Step 7 — Verdict & Structured Report

### Verdict Definitions
- **INSTALL** — Safe, relevant, ready to install.
- **REVISIT** — Safe but not currently relevant, or sandbox testing needed first. Always include a specific trigger condition for when to revisit.
- **SKIP** — Safe but not relevant to current work or stack.
- **REJECT** — Security concerns found. Do not install. Explain what was found.

### Report Format

```
## Repo Review: [repo-name]
**URL:** [url]
**Date:** [today's date]
**Verdict:** INSTALL / REVISIT / SKIP / REJECT

### Security
[findings — hard rejects, yellow flags, Claude Code-specific checklist, runtime trust surface gap analysis]

### Relevance
[High / Medium / Low / None — with specific reasoning, or "Skipped — no user context configured"]

### Overlap with Existing Skills
[comparison if applicable, or "None" if no overlap found]

### Recommendation
[clear next step — install now, sandbox first, revisit when X, or do not install]
```

## Step 8 — Review Log

After delivering the report:

1. **If the user has an existing review log:** Ask if they'd like to append this entry to it.
2. **If no review log has been mentioned:** Ask:
   > "Would you like to start a review log to track all future repo reviews? I can create a simple markdown file to record URL, date, and verdict for every repo you evaluate."
   If confirmed, create `review-log.md` in a location the user specifies, or suggest the skill's own folder (`~/.claude/skills/repo-review/review-log.md`) as a sensible default. Format it with a header and this review as the first entry.
