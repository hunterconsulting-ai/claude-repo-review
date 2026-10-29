# The review log — index, detail, delta reviews, revisit sweeps

Two files, because a log that grows past what can be read in one pass stops answering "have we reviewed this?":

| File | Holds | Read how |
|---|---|---|
| **Index** (`review-index.md`) | One row per review | Grep it in Step 1 — print only the id, date, repo and commit columns. Small enough to stay cheap forever. |
| **Log** (`review-log.md`) | The full report of each review, appended | Open one entry when the index points to it — after your own verdict on a fresh review. Never read it whole. |

Paths come from the user context; defaults are `~/.claude/repo-review/`. Keep both outside the skill's folder. **If the user context defines its own row or heading format, or its own numbering, that wins over everything below.**

## Index row (default format)

```
| # | Date | Repo | Commit | Verdict | Revisit when | Log |
|---|---|---|---|---|---|---|
| 42 | 2026-10-28 | owner/repo | 1a2b3c4 | REVISIT | the n8n build starts | review-log.md#42 |
```

- **Repo** is `owner/name` — the pre-check greps it. For a satellite, the code repo, with the satellite noted. For a local folder, the folder name plus `(local folder)` — never a claimed `owner/name`, which the report records as a claim. For a remote MCP endpoint, the endpoint host.
- **Commit** is the SHA reviewed — a local folder's full 64-hex content hash, `n/a (hosted)` for a remote endpoint (the Date column is what its verdict covers), `—` for entries logged before v2.
- **Revisit when** holds a REVISIT's trigger, or a REJECT's reconsider-when condition. It must be checkable.
- **Numbering:** one past the **highest** number in that log (not the last row — logs get out of order). With several logs in one index, number within each log.
- **Later records win.** A new review of the same repo and scope supersedes the old row's verdict; say which entry it supersedes.

## Log entry

```
## #42 — owner/repo
[the full Step 8 report, from its **URL:** line down — this heading replaces the report's own "## Repo Review" title]
```

## Delta review — "is the update to X safe?"

For a repo already reviewed at a known commit. A delta reports against the old verdict, so it opens the old entry first — Ground rule 4 covers fresh reviews.

1. Get the change set from the reviewed commit to the current head: `GET /repos/{owner}/{repo}/compare/{reviewed_sha}...{branch}` (or `git diff <sha>..HEAD` in a scratch clone).
2. Review **what changed**, with the full method's eyes: new or changed payload files (SKILL.md, agents, commands, hooks, manifests, `.mcp.json`), new dependencies (and do they exist on their registry?), new network calls, a changed license file, a broadened trigger description.
3. **When the change calls into existing code, read the code it calls.** A payload can be split across updates — a dormant function in one, the call that wakes it in the next — and a diff shows only the second half.
4. For text-only skills, the high-value checks are mechanical: hidden or zero-width characters, new URLs, new shell commands, text addressed to the agent.
5. Report against the old verdict: *unchanged / changed because …*. Log it as a new row with the new commit.
6. **No commit on the old entry** (pre-v2): there is nothing to diff against — the same two options as Step 1: run a full review, or compare the installed copy with upstream by per-file hashes.
7. **A local folder:** hash the new copy the same way. Same hash = same code. Otherwise diff its per-file hash list against the one kept from the last review; with no list kept, run a full review.
8. **What a comparison may show** (installed vs upstream, old vs new): compare per-file hashes first. Show text diffs only for code and docs, redacting any credential a diff would print (Ground rule 5). For config, env and credential files, report "differs" plus the changed key **names** only — never values (Ground rule 5).

This makes "is the upgrade worth it?" a short review of the diff instead of a full re-review.

## Revisit sweep — "sweep the revisits"

1. Pull every REVISIT row from the index, plus REJECT rows with a reconsider-when condition.
2. For each, check whether its trigger has fired — ask the user when only they can know.
3. Fired → offer a delta or fresh review. You have just read the old verdict, so a fresh review runs in a **fresh subagent** briefed without it (as in [batch-reviews.md](batch-reviews.md)). Not fired → leave it. No checkable trigger → write one now, with the user.
4. Report a short table: repo · trigger · fired? · next step.

## Moving a v1 log to v2

A v1 log is one growing file. **First move it out of the skill folder** — v1's default location was inside it, and updating the skill replaces that folder. Don't rewrite it — **build an index from it** and list the old file as an archive log in the user context. New reviews go to the v2 files. Real v1 logs are messier than they look; a parser that trusts the first match gets verdicts wrong. Measured on 636 entries across three logs:

- **Entries come in several shapes** — numbered headings, batch tables, resolution tables with a "final verdict" column. Index all of them and let **later records win**.
- **Resolution lines inside an entry's body change its verdict** ("Update: …", "Resolved: …", sometimes far down the file). Order records by date, not by position in the file.
- **Some reviews have no number.** Give each its own row, keyed by date and repo, rather than folding it into a neighbour.
- **A table of separately judged items inside one entry** (several repos or skills, each with its own verdict) is one row per item, not one row for the entry.
- **Verdict lines get edited in place:** `~~REVISIT~~ → INSTALL`, `REVISIT → ✅ INSTALLED`. Drop struck text and follow an arrow — **but only to another verdict word**: `REVISIT (HeyGen → n8n node)` is still a REVISIT.
- **Split verdicts** (`SKIP / REVISIT when …`) keep their open half: index them as the REVISIT, with its trigger, so the sweep still finds them.
- **Legacy verdict words** (`EVALUATE`, `BOOKMARK`, `N/A`) lead the line; keep them verbatim rather than searching the rest of the line for a familiar word.
- **The trigger field is spelled many ways** — "Revisit trigger", "Conditions to revisit", "Trigger to revisit", plural headings, plain prose ("Revisit … when …"), a table's notes column. Search all of them, case-insensitively.
- **"Reason for REVISIT" is not the trigger.** It says why the decision was deferred, not what to check. An entry with only a reason is a REVISIT without a trigger.
- **Later records win across logs, not just within one** — a review in one log can be superseded by one in another. Add a supersession pointer only when the scope matches: the same repo, and the same component of it.
- **`→` means two things** — a verdict transition (`REVISIT → INSTALL`) and a path separator (`Settings → Plugins`). An open-items sweep needs a column-aware command (split the row, read the Verdict column), not a line grep.
- **Numbers collide** — the same number reused for two different repos. Split those into separate rows rather than merging one repo's verdict onto another.
- **Verify before trusting it:** row count = entry count per log; spot-check rows you know; list REVISIT rows left without a trigger; diff verdicts against a previous build after any parser change.
