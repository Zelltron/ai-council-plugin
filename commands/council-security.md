---
description: Security-focused council review — trust boundaries, injection, unsafe assumptions
argument-hint: "[--branch=main|--pr=123|--staged|--deep] [extra context]"
allowed-tools: ["mcp__plugin_ai-council_ai-council__security", "mcp__plugin:ai-council:ai-council__security"]
---

Run the `security` tool from the bundled `ai-council` MCP server.

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

These agents are also asked what looks risky but is acceptable, and what should
not be hardened yet. Include that — it is as useful as the findings, and it
keeps the review from turning into busywork.

Verify each finding against the actual code before endorsing it. Security
reviewers working from a diff alone routinely flag parameterized queries as
injection and miss authentication applied at a layer they cannot see. Say which
findings you confirmed and which you could not.
