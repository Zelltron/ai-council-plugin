---
description: Architecture review board — is the abstraction justified, what hurts in 6-12 months
argument-hint: "[--branch=main|--pr=123|--staged|--deep] [design question or context]"
allowed-tools: ["mcp__plugin_ai-council_ai-council__arch", "mcp__plugin:ai-council:ai-council__arch"]
---

Run the `arch` tool from the bundled `ai-council` MCP server.

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
| anything else | `text` |

This is the one council command that is often worth running with no diff at all.
If the user describes a design they are considering rather than code they have
written, pass their description as `text` and leave the git parameters unset.

With no arguments at all, pass only `cwd` to review the working tree.

## Reporting the result

Report the verdict and confidence, the judge's rationale, and each agent's vote,
highlighting dissent.

These agents answer three questions directly: what the architectural risk is,
what should not be changed, and what assumptions must hold for the design to be
correct. Report all three. The assumptions matter most — they are the part a
future reader cannot reconstruct from the code.

If the council proposes a simpler alternative, state it and say whether it is
actually compatible with constraints you can see in the codebase.
