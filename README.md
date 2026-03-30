# claude-repo-review

A Claude Code skill for reviewing GitHub repos before you install them — covering security, relevance to your work, and overlap with tools you already have.

Built for Claude Code. Platform agnostic by design — works across all Claude Code surfaces: CLI, VS Code extension, JetBrains extension, desktop app, and claude.ai/code.

---

## What It Does

Paste a GitHub URL and ask for a review. The skill runs a structured 8-step analysis:

1. Checks your review log for prior verdicts (optional — see [Review Log](#review-log))
2. Fetches the repo's README, package manifests, install scripts, and license
3. Security analysis — hard rejects, yellow flags, runtime trust surface
4. Relevance to your work (if you've configured a user context — see [Customization](#customization))
5. Overlap check against skills you already have installed
6. Sandbox recommendation when the verdict is inconclusive
7. Structured report with a clear verdict: **INSTALL / REVISIT / SKIP / REJECT**
8. Offers to log the result or start a review log

---

## Install

```bash
# Clone the repo
git clone https://github.com/hunterconsulting-ai/claude-repo-review.git

# Copy the skill into your Claude Code skills directory
cp -r claude-repo-review ~/.claude/skills/repo-review
```

Or manually: copy `SKILL.md` (and optionally `user-context.example.md`) into a folder named `repo-review` inside `~/.claude/skills/`.

> **Windows users:** Your skills directory is typically `C:\Users\<you>\.claude\skills\`.

---

## Usage

In any Claude Code session, paste a GitHub URL and ask:

> "Review this repo before I install it: https://github.com/owner/repo"

The skill triggers automatically based on the request pattern. No slash command required, though `/repo-review` also works if your setup supports it.

---

## Customization

The relevance assessment step (Step 4) evaluates repos against your specific work context. Without configuration it is skipped — the rest of the analysis still runs in full.

To enable it:

1. Copy `user-context.example.md` to `user-context.md` in the same skill folder
2. Fill in your role, services, tech stack, and any environment-specific security concerns
3. Save — the skill reads it automatically on the next review

The skill will also prompt you during a session if no context file is found.

---

## Review Log

After each review, the skill offers to log the result. If you don't have a log yet, it will offer to create one for you — a simple markdown file that records repo URL, date, and verdict. Useful for auditing what you've evaluated over time.

The default location is `~/.claude/skills/repo-review/review-log.md`, but you can specify any path.

---

## Platform Note

This skill uses Claude Code's `SKILL.md` format. It is not tied to any specific Claude model, IDE, or operating system. Any Claude Code environment that supports custom skills can run it — including the CLI, VS Code extension, JetBrains extension, desktop app, and claude.ai/code.

---

## License

MIT
