# AI Council — Claude Code plugin

Multi-agent code review for Claude Code. Five specialized agents vote APPROVE,
REVISE, or REJECT on a change and a judge synthesizes the verdict. When the
Gemini agent dissents from the majority, a debate round runs before the ruling.

The point is not the verdict. It is that the panel did not write the code, so it
has no stake in defending it. Dissent between agents is signal, not noise.

## What is in this repository

Only the Claude Code integration: a manifest, an MCP server declaration, five
slash commands, and two skills. Roughly a dozen text files.

The review engine itself ships separately as the
[`@mugzie/ai-council`](https://www.npmjs.com/package/@mugzie/ai-council) npm
package, which the plugin launches on demand. Nothing here needs building.

## Install

```bash
claude plugin marketplace add Zelltron/ai-council-plugin
claude plugin install ai-council@ai-council
```

Restart Claude Code, then confirm the server connected:

```
/mcp
```

You should see `ai-council` listed as connected. If anything is missing, ask
Claude to set up AI Council — the bundled setup skill diagnoses it and walks you
through the fix.

## Requirements

Set these in the shell you launch Claude Code from. The MCP server inherits the
environment; no keys are stored in the plugin.

| Variable | Required | Purpose |
|---|---|---|
| `AI_COUNCIL_LICENSE_KEY` | Yes | License key from [ai-council.lemonsqueezy.com](https://ai-council.lemonsqueezy.com) |
| `OPENAI_API_KEY` | One of the two | Primary agents |
| `GEMINI_API_KEY` | One of the two | Gemini agent and the dissent/debate round |
| `AI_COUNCIL_MODEL` | No | OpenAI model, default `gpt-5.4-mini` |
| `AI_COUNCIL_GEMINI_MODEL` | No | Gemini model, default `gemini-3.6-flash` |
| `GH_TOKEN` | For private PRs | `--pr` reviews shell out to `gh` |

Configure both API keys if you can. The debate round only has teeth when the
panel spans two providers — with one, there is no independent perspective to
dissent and the panel converges too easily.

Node is required. The server runs via `npx -y @mugzie/ai-council mcp`, which
resolves the prebuilt binary for your platform. No global install needed.

Verify your keys reach the engine at any time:

```bash
npx -y @mugzie/ai-council test
```

## Commands

| Command | Focus |
|---|---|
| `/council-review` | Quality, hidden risk, unnecessary complexity |
| `/council-security` | Trust boundaries, injection, unsafe assumptions |
| `/council-perf` | CPU, allocation, render frequency, async and IO |
| `/council-arch` | Whether the abstraction is justified, and its 6-12 month cost |
| `/council-sanity` | Whether the work is over-engineered |

All five accept the same diff selectors:

```
/council-review
/council-review --branch=main
/council-review --pr=123 --deep
/council-security --staged
/council-arch I'm considering moving graph discovery behind an interface
```

With no arguments the working tree is reviewed, preferring staged over unstaged
changes. If your work is already committed the tree is clean, so use
`--branch=main`.

`--deep` adds file-by-file findings with line numbers. `--agent` lets the council
explore the codebase with its own Read and Grep, for changes whose risk lives in
code the diff does not show. Both cost noticeably more.

## Skills

`council-review` lets Claude reach for the council on its own when you are about
to commit or merge, and — just as importantly — tells it when not to. Commands
only fire when you type them, which is why MCP-only setups tend to go unused.

`ai-council-setup` handles first-run configuration and diagnoses a council that
is not working.

## Deliberately not included

There is no `Stop` or `PostToolUse` hook. A hook would run a paid multi-agent
review on every turn against your own API keys, whether or not the diff changed.
Hooks also load only at session start, so tuning one means restarting Claude Code
each time.

If you want that tradeoff, add `hooks/hooks.json`:

```json
{
  "Stop": [{
    "matcher": ".*",
    "hooks": [{
      "type": "command",
      "command": "npx -y @mugzie/ai-council review --diff --ci",
      "timeout": 120
    }]
  }]
}
```

`--ci` exits non-zero on REJECT or a low-confidence REVISE. Gate it behind a
`.claude/ai-council.local.md` settings file so it stays opt-in per project.

## Troubleshooting

**Server missing from `/mcp`** — plugins load at session start, so restart Claude
Code after installing. Then confirm `AI_COUNCIL_LICENSE_KEY` is exported in the
launching shell.

**Verdict is `NO_CHANGES`** — the selected diff was empty. Not a failure. Use
`--branch=main` if the work is already committed.

**Commands prompt for permission every time** — Claude Code has used more than
one naming scheme for plugin-provided MCP tools, so each command pre-allows both
`mcp__plugin_ai-council_ai-council__<tool>` and
`mcp__plugin:ai-council:ai-council__<tool>`. If your version uses a third,
compare against `/mcp` and add it to the command's `allowed-tools`. An unmatched
entry is harmless; a miss only costs you the prompt.

**Review targets the wrong repository** — the commands pass `cwd` explicitly. If
it is still wrong, relaunch Claude Code from the repository root.

## Other ways to use AI Council

The same engine is available as a standalone CLI and as a Cursor extension. See
[ai-council.lemonsqueezy.com](https://ai-council.lemonsqueezy.com).

## License

Two different licenses apply, so read this before forking.

The **Claude Code integration in this repository** — the manifest, MCP server
declaration, slash commands, skills, and documentation — is licensed under
Apache-2.0. See [LICENSE](LICENSE).

The **AI Council review engine** is not. It ships separately as the
[`@mugzie/ai-council`](https://www.npmjs.com/package/@mugzie/ai-council) npm
package under a proprietary license, requires a valid paid license key, and may
not be copied, modified, redistributed, or reverse engineered.

Nothing in this repository grants any right to the engine, and Apache-2.0 grants
no rights to the "AI Council" name or marks. See [NOTICE](NOTICE) for the
authoritative scope statement, which redistributors are required to preserve.
