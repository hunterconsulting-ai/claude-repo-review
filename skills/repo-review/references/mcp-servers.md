# MCP servers — the trust-surface module

Load this when the target is an MCP server. It **adds** to the main review: Steps 0–3 and the rest of Step 4 still apply. An MCP server is code that Claude calls on its own initiative, with arguments Claude composes from whatever it has read — so what each tool can do with a malicious argument matters more than what the README says it is for.

## Extra things to gather

- The source file(s) that register tools, resources and prompts.
- Every HTTP handler, fetch or network call; all authentication and credential handling.
- For a hosted endpoint: the vendor's privacy policy and terms — what they log from requests, and whether queries train models.

## Extra hard rejects and yellow flags

- **Hard reject:** a tool that executes arbitrary caller-supplied strings as shell commands without sandboxing.
- **Yellow:** no validation on tool parameters (prompt injection flows straight into tool calls) · HTTP/SSE transport with no authentication · unbounded scope (whole file system, unrestricted network).

## The trust surface — document each

1. **Transport.**
   - *stdio* — a local child process Claude launches; not callable from the network. **Lowest network exposure — but it runs as the user, with their files and environment variables.** Its code is the risk; read all of it.
   - *HTTP / SSE* — a network server. Does it bind to `127.0.0.1` or to every interface? Unauthenticated + all interfaces = anything on the network can call it.
   - *Remote endpoint* (`mcp.vendor.com`) — every call goes through a third party. **Trust sits with the vendor, not with auditable code**; treat it like a third-party API and say so.
2. **Tools.** For each: name, what it does, its parameters (flag free-form strings, file paths and URLs), whether it reads / writes / executes / calls out, and its **blast radius** — the worst outcome if called with malicious input.
3. **Resources.** What data it exposes, read-only or writable, anything sensitive (credentials, PII, internal state).
4. **Prompts.** What templates it registers, and whether they inject fixed instructions into the conversation.
5. **Authentication.** Required? How credentials are stored and passed (env var, config file, hardcoded)? Without credentials, does it fail closed or open? **Is TLS certificate verification on** for the calls that carry the credential (`verify=False` sends it to whoever answers)?
6. **Outbound calls.** Which tools call out, to which endpoints, whether user input or Claude's output is sent, and over HTTPS?
7. **File-system scope.** What it reads and writes, whether access is confined to a directory, and what it stores locally.
8. **What the backend can be made to do.** If the server wraps another API (a local REST plugin, a SaaS), list that API's **dangerous endpoints**, then check whether a tool parameter can reach them. A file path pasted unencoded into a URL can climb out with `..` — HTTP clients collapse it — and land on a different endpoint entirely. (`mcp-obsidian` pastes vault paths into `/vault/{path}` unvalidated — upstream issue #150, path traversal — and the plugin behind it also exposes an *execute a command* endpoint.)

## Overlap

Compare against the MCP servers **actually connected** — the session's tool list, or the `mcpServers` entries in `~/.claude.json` and any `.mcp.json` (including servers disabled for a project, which count as owned) — never a remembered list. Many servers duplicate one another, and a capability may already be covered by a connector the user has.

## Verdict

Use **CONNECT** in place of INSTALL and give the registration command:

```bash
# stdio — everything after -- goes to the server untouched
claude mcp add --scope user <name> -- <command> [args...]
# HTTP
claude mcp add --scope user --transport http <name> <url>
```

🔑 **Credentials:** `claude mcp add --env KEY=value` stores the **literal** value in config. To keep secrets out of config files, put the server in a `.mcp.json` entry and reference them as `${VAR}` (or `${VAR:-default}`) in `env`, `args`, `url` or `headers` — Claude Code expands them at launch. An unset variable still loads, with the raw `${VAR}` text; `claude mcp list` warns about it.

Report the transport and the tool list on the report's **Transport / tools** line. If the README's install command is unpinned (`uvx <server>`), give a pinned one — `uvx` ignores the repo's lockfile.
