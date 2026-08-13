---
description: Sanity check — is this over-engineered, what is the simplest acceptable version
argument-hint: "[--branch=main|--pr=123|--staged] [what you are second-guessing]"
allowed-tools: ["mcp__plugin_ai-council_ai-council__sanity", "mcp__plugin:ai-council:ai-council__sanity"]
---

Run the `sanity` tool from the bundled `ai-council` MCP server.

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
| anything else | `text` |

With no arguments, pass only `cwd` to review the working tree.

## Reporting the result

These agents are asked to answer brutally honestly whether the work is
over-engineered, what the simplest acceptable version is, and what problem the
change is really solving. They are also asked to say when the change should be
left alone or abandoned outright.

Report that verdict without softening it. If the council says the change should
be abandoned and you disagree, say so and give your reason — but do not quietly
downgrade "abandon this" to "consider simplifying".

Report the verdict, confidence, judge rationale, and agent votes, highlighting
dissent.
