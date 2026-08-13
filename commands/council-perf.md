---
description: Performance-focused council review — CPU, allocation, render frequency, async and IO
argument-hint: "[--branch=main|--pr=123|--staged|--deep] [extra context]"
allowed-tools: ["mcp__plugin_ai-council_ai-council__perf", "mcp__plugin:ai-council:ai-council__perf"]
---

Run the `perf` tool from the bundled `ai-council` MCP server.

Arguments: $ARGUMENTS

## Building the tool call

Translate the arguments into tool parameters. Always pass `cwd` set to the
absolute path of the repository root.

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

With no arguments, pass only `cwd` to review the working tree.

`pr` cannot be combined with `gitBranch`, `gitCommit`, `gitRange`, or
`reviewBranch`.

## Reporting the result

Report the verdict and confidence, the judge's rationale, and each agent's vote,
highlighting dissent. Group `fileReviews` findings by file with line numbers.

These agents are asked to say where performance degrades under scale and whether
an optimization is premature. Lead with the scale claim: a finding that only
matters at a load this code will never see is not worth acting on, and the
council is explicitly asked to say so. Report that judgment, not just the list.

Do not report stylistic refactors as performance findings.
