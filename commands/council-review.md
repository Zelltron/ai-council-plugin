---
description: Multi-agent code review of your changes — quality, hidden risk, unnecessary complexity
argument-hint: "[--branch=main|--pr=123|--staged|--deep|--suggestions] [extra context]"
allowed-tools: ["mcp__plugin_ai-council_ai-council__review", "mcp__plugin:ai-council:ai-council__review"]
---

Run the `review` tool from the bundled `ai-council` MCP server.

Arguments: $ARGUMENTS

## Building the tool call

Translate the arguments into tool parameters. Always pass `cwd` set to the
absolute path of the repository root so the server resolves the right git
repository.

| Argument | Parameter |
|---|---|
| `--branch=X`, "vs X", "against X" | `gitBranch: "X"` |
| `--pr=N` or a GitHub pull request URL | `pr: "N"` |
| `--staged`, `--unstaged`, `--all` | `gitScope` |
| `--commit=HASH` | `gitCommit: "HASH"` |
| `--range=A..B` | `gitRange: "A..B"` |
| `--deep` | `deepAnalysis: true` |
| `--agent` | `agent: true` |
| `--suggestions`, "with fixes" | `suggestions: true` |
| anything else | `text` |

With no arguments, pass only `cwd`. The server then reviews the working tree,
preferring staged over unstaged changes.

`pr` cannot be combined with `gitBranch`, `gitCommit`, `gitRange`, or
`reviewBranch`. If the user asks for such a combination, tell them and pick one.

## Reporting the result

The tool returns a `CouncilResult` as JSON. Do not paste the raw JSON. Report:

1. The verdict (`finalAnswer`) and confidence as a percentage.
2. The judge's rationale, in your own words if it is long.
3. Each agent's vote, and call out any agent that dissented from the majority —
   a dissent is the most informative part of the result.
4. Findings from `fileReviews` when present, grouped by file with line numbers.

Then say plainly whether you agree. You have read the code; the council has only
read the diff, so you are well placed to flag a finding that misses context.
Do not act on the findings unless the user asks.

If `finalAnswer` is `NO_CHANGES`, there was no diff to review. Say so and
suggest `--branch=main` if the work is already committed.
