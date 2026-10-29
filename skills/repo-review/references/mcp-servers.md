# MCP servers — the trust-surface module

Load this when the target is an MCP server — or an application that embeds one, for that component. It **adds** to the main review: Steps 0–9 all still apply. An MCP server is code that Claude calls on its own initiative, with arguments Claude composes from whatever it has read — so what each tool can do with a malicious argument matters more than what the README says it is for.

## Extra things to gather

- The source file(s) that register tools, resources and prompts.
- Every HTTP handler, fetch or network call; all authentication and credential handling.
- For a hosted endpoint: the vendor's privacy policy and terms — what they log from requests, and whether queries train models.

## Extra hard rejects and yellow flags

- **Hard reject:** a tool that executes arbitrary caller-supplied strings as shell commands without sandboxing.
- **Yellow:** no validation on tool parameters (prompt injection flows straight into tool calls) · HTTP/SSE transport with no authentication — even on `127.0.0.1`, any process on the host can call it · unbounded scope (whole file system, unrestricted network) · a disclosed vendor-hosted endpoint that *is* the product (see *Transport*).

## The trust surface — document each

1. **Transport.**
   - *stdio* — a local child process Claude launches; not callable from the network. **Lowest network exposure — but it runs as the user, with their files and environment variables.** Its code is the risk; read all of it.
   - *HTTP / SSE* — a network server. Does it bind to `127.0.0.1` or to every interface? Unauthenticated on `127.0.0.1` = any process on the host can call it; unauthenticated + all interfaces = anything on the network can.
   - *Remote endpoint* (`mcp.vendor.com`) — every call, and whatever Claude puts in it, goes to a third party. **Trust sits with the vendor, not with auditable code**; treat it like a third-party API and say so. A disclosed endpoint that *is* the product is a yellow flag weighed in the verdict, not a hard reject (SKILL.md Step 4); user content sent anywhere unrelated to the tool's stated function still is one — including telemetry that carries user content, unless it is opt-in *and* disclosed before install (SKILL.md Step 4 defines both). There is no commit: log **Repo** = the endpoint host and **Commit** = `n/a (hosted)` — the review date is what the verdict covers.
2. **Tools.** For each: name, what it does, its parameters (flag free-form strings, file paths and URLs), whether it reads / writes / executes / calls out, and its **blast radius** — the worst outcome if called with malicious input.
3. **Resources.** What data it exposes, read-only or writable, anything sensitive (credentials, PII, internal state).
4. **Prompts.** What templates it registers, and whether they inject fixed instructions into the conversation.
5. **Authentication.** Required? How credentials are stored and passed (env var, config file, hardcoded)? Without credentials, does it fail closed or open? **Is TLS certificate verification on** for the calls that carry the credential (`verify=False` sends it to whoever answers)?
6. **Outbound calls.** Which tools call out, to which endpoints, whether user input or Claude's output is sent, and over HTTPS?
7. **File-system scope.** What it reads and writes, whether access is confined to a directory, and what it stores locally.
8. **What the backend can be made to do.** If the server wraps another API (a local REST plugin, a SaaS), list that API's **dangerous endpoints**, then check whether a tool parameter can reach them. A file path pasted unencoded into a URL can climb out with `..` — HTTP clients collapse it — and land on a different endpoint entirely. (`mcp-obsidian` pastes vault paths into `/vault/{path}` unvalidated — upstream issue #150, path traversal — and the plugin behind it also exposes an *execute a command* endpoint.)

## Relevance and overlap

**Relevance (Step 5):** don't stretch for it — if the server fits none of the user's current work, say so plainly.

**Overlap:** compare against the MCP servers **actually connected** — the session's tool list, or the server **names** in the `mcpServers` entries of `~/.claude.json` and any `.mcp.json` (parse the keys only — never print the file whole, or any `env` or `headers`, which can hold literal credentials), including servers disabled for a project, which count as owned — never a remembered list. Many servers duplicate one another, and a capability may already be covered by a connector the user has. Where it is, is the new server **more capable, better maintained, or offering meaningfully different tools**? Recommend **keep existing / replace / connect alongside**.

## Verdict

Use **CONNECT** in place of INSTALL and give the registration command, with the scope from the user context's install defaults (else `user`):

```bash
# stdio — everything after -- goes to the server untouched
claude mcp add --scope <scope> <name> -- <command> [args...]
# HTTP
claude mcp add --scope <scope> --transport http <name> <url>
```

🔑 **Credentials:** `claude mcp add --env KEY=value` stores the **literal** value in config. To keep secrets out of config files, reference them as `${VAR}` (or `${VAR:-default}`) in `command`, `args`, `env`, `url` or `headers` — Claude Code expands them at launch, in every scope. For the default user scope, register the entry as JSON, single-quoted so the shell leaves `${VAR}` alone: `claude mcp add-json --scope user <name> '<json with ${VAR}>'` (Windows PowerShell 5.1 strips embedded double quotes from native-command arguments — run it from another shell, or check the stored entry). A `.mcp.json` entry also works, but that is **project** scope. An unset variable still loads, with the raw `${VAR}` text; `claude mcp list` warns about it.

If the README's install command is unpinned (`uvx <server>`), give a pinned one — `uvx` ignores the repo's lockfile. Some clients pick up a newly registered server only after a full restart, not just a new conversation — say so if it may apply.

## Report block

Report the transport and the tool list on the report's **Transport / tools** line, and fill the **MCP trust surface** section:

```
### MCP trust surface
**Tools:** [each: name · reads / writes / executes / calls out · blast radius]
**Resources:** [list, or "None"]
**Prompts:** [list, or "None"]
**Authentication:** [method · where credentials live · fails open or closed · TLS verification]
**Outbound:** [endpoints, per tool · what is sent · HTTPS?]
**Filesystem:** [what it reads and writes, and where · confined to a directory?]
```
