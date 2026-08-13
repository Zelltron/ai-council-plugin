---
name: ai-council-setup
description: Use when AI Council is not working — the user has just installed the plugin, the council tools are missing from the session, a review fails with a license error or "No license key provided", an API key is missing or rejected, or the user asks how to configure, set up, or authenticate AI Council. Walks through license and API key setup and verifies the MCP server end to end.
version: 0.1.0
---

# Setting up AI Council

AI Council needs three things: Node on the PATH, a license key, and at least one
model API key. All of them come from the environment of the shell that launched
Claude Code — nothing is stored in the plugin.

Work through this in order and stop at the first step that fails. Most setup
problems are the last step, not the first.

## 1. Identify what is actually wrong

Ask the user to run `/mcp` and tell you what they see, or check it yourself.

| Symptom | Cause | Go to |
|---|---|---|
| No `ai-council` server listed at all | Plugin not installed or not enabled | Step 2 |
| Server listed but failed to connect | Node or npx cannot run | Step 3 |
| Server connected, review fails with a license message | No or invalid license key | Step 4 |
| Review fails naming OpenAI or Gemini | No or invalid model API key | Step 5 |
| Everything connects, review returns `NO_CHANGES` | Setup is fine; there is just no diff | Step 7 |

Do not move keys around before you know which of these it is. A connected server
with a license error is a completely different fix from a server that never
started.

## 2. Confirm the plugin is installed and enabled

```bash
claude plugin list
```

Look for `ai-council` with status enabled. If it is missing:

```bash
claude plugin marketplace add Zelltron/ai-council-plugin
claude plugin install ai-council@ai-council
```

Plugins and their MCP servers load at session start, so the user must restart
Claude Code after installing. Say this explicitly — it is the most common reason
a correct install appears broken.

## 3. Confirm the server can start

The plugin runs the server with `npx -y @mugzie/ai-council mcp`. Verify Node and
the package resolve:

```bash
node --version
npx -y @mugzie/ai-council --version
```

If Node is missing, that is the fix. The package is public on npm and needs no
registry authentication, so a 401 or 404 here means a proxy or registry override,
not a missing entitlement.

## 4. Set the license key

AI Council requires a paid license from
[ai-council.lemonsqueezy.com](https://ai-council.lemonsqueezy.com).

```bash
export AI_COUNCIL_LICENSE_KEY="..."
```

## 5. Set at least one model API key

```bash
export OPENAI_API_KEY="..."
export GEMINI_API_KEY="..."
```

One is enough to run. Recommend both anyway, and give the real reason: the
council escalates to a debate round when the Gemini agent dissents from the
majority. With a single provider there is no independent perspective to dissent,
so the panel converges more readily and the output is worth less.

Optional overrides, both with working defaults:

- `AI_COUNCIL_MODEL` — OpenAI model, defaults to `gpt-5.4-mini`
- `AI_COUNCIL_GEMINI_MODEL` — Gemini model, defaults to `gemini-3.6-flash`

For `--pr` reviews on private repositories, the `gh` CLI must be authenticated or
`GH_TOKEN` must be set.

## 6. Make the variables persist, then restart

Exports in one terminal do not reach a Claude Code session started from another,
and this is where setup usually goes wrong. Have the user add the exports to
their shell profile — `~/.zshrc` on macOS — then start a new terminal and
relaunch Claude Code.

Verify the keys reach the engine:

```bash
npx -y @mugzie/ai-council test
```

That checks both providers and reports which are configured and reachable. Treat
it as the source of truth over any assumption about what is exported.

Never write keys into `.mcp.json`, a committed settings file, or anything else
that lands in git. If the user wants per-project keys instead of global ones, an
`env` block in `.mcp.json` may reference variable *names* only.

## 7. Confirm it works end to end

Run a real review against a branch with actual changes:

```
/council-review --branch=main
```

A verdict with agent votes means setup is complete.

`NO_CHANGES` is not a setup failure. It means the selected diff was empty —
usually because the work is already committed and the working tree is clean. Use
`--branch=main` in that case.
