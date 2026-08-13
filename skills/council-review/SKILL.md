---
name: council-review
description: Use when the user wants a second opinion on a change before shipping — asks to "review this", "get a review", "run the council", "is this safe to merge", "is this over-engineered", "check this for security/performance/architecture problems", or wants independent verification of a diff, commit, branch, or pull request. Routes to the AI Council MCP tools, which run several specialized agents and a judge over the diff.
version: 0.1.0
---

# AI Council Review

AI Council runs a panel of specialized agents over a diff. Each votes APPROVE,
REVISE, or REJECT with a confidence score, and a judge synthesizes a single
verdict. When the Gemini agent dissents from the majority, a debate round runs
before the judge rules.

The value is not the verdict. It is that the panel is independent of you: it did
not write the code, so it has no stake in defending it. Dissent between agents is
signal, not noise.

## When to use it

Reach for the council when a change is about to leave your hands:

- The user is about to commit, merge, or open a pull request.
- The user asks whether a change is safe, correct, or worth keeping.
- You have just finished a non-trivial change and want verification that is not
  your own judgment.
- The user is second-guessing scope or complexity.

## When not to use it

Every run costs several LLM calls against the user's own API keys and takes tens
of seconds. Skip it for:

- Typos, formatting, renames, dependency bumps.
- Work in progress that the user is still actively changing.
- Anything you can verify yourself by reading the code or running tests. Run the
  tests first; a failing test is better evidence than six opinions.
- Repeat runs on a diff that has not changed since the last review.

Do not run more than one agent set on the same diff unless the user asks. Pick
the one that matches the question.

## Choosing the agent set

| Tool | Use for |
|---|---|
| `review` | General quality, hidden risk, unnecessary complexity. The default. |
| `security` | Trust boundaries, injection, serialization, unsafe assumptions, silent failures. |
| `perf` | CPU, allocation and retention, render frequency, async and IO. |
| `arch` | Whether an abstraction is justified and what it costs in 6-12 months. |
| `sanity` | Whether the work is over-engineered and what the simplest version is. |

`decide` takes an explicit `agentSet` plus a free-form `question`. Use it only
when you need a question the five prompts do not cover; otherwise the named
tools are clearer.

`arch` and `sanity` are useful with no diff at all — pass a design description as
`text` when the user is weighing an approach they have not built yet.

## Selecting the diff

Always pass `cwd` as the absolute path of the repository root. The MCP server
runs as a separate process and its working directory is not guaranteed to match
the project.

Then select exactly one source:

- Nothing set: the working tree, preferring staged over unstaged changes.
- `gitScope`: `staged`, `unstaged`, or `all` to override that preference.
- `gitBranch: "main"`: current HEAD against a branch. Use this when the work is
  already committed — otherwise the working tree is clean and there is nothing
  to review.
- `pr`: a pull request number or URL. Requires the `gh` CLI, and `GH_TOKEN` for
  private repositories. Cannot be combined with `gitBranch`, `gitCommit`,
  `gitRange`, or `reviewBranch`.
- `gitCommit`, `gitRange`, or `reviewBranch` with `reviewBranchBase`.

## Depth options

Leave these off by default. Each one multiplies cost and latency.

- `deepAnalysis: true` — file-by-file review with line-level structured findings
  in `fileReviews`. Worth it for a substantial change the user is about to merge.
- `agent: true` — two-pass review where the council explores the codebase with
  its own Read and Grep. Implies `deepAnalysis`. Best for changes whose risk
  lives in code the diff does not show. Requires
  `@anthropic-ai/claude-agent-sdk` to be installed.
- `suggestions: true` — agents include fix examples when they say REVISE or
  REJECT. These are AI-generated and unverified; treat them as illustrations.
- `preContext: true` — injects AST-extracted, embedding-ranked code context.
- `graph: true` / `noGraph: true` — force graph-neighborhood context on or off.
  Discovery is automatic and fails open, so you rarely need either.

## Reading the result

The tools return a `CouncilResult`. Never paste the raw JSON at the user. Report:

1. `finalAnswer` and `confidence` as a percentage. Below 65% the CLI treats the
   result as inconclusive; say so rather than presenting a weak verdict as firm.
2. `judgeRationale`, summarized if long.
3. `votes` — each agent's answer and confidence. Always surface a dissent and
   name the agent. A lone REJECT against four APPROVEs is the single most useful
   line in the output.
4. `fileReviews` when present, grouped by file with line numbers.

`finalAnswer: "NO_CHANGES"` means no diff was found, not that the code is clean.
Check whether the changes are already committed and retry against a branch.

## After the review

You have context the council does not: it saw a diff, you have read the
codebase. Verify findings before endorsing them. Reviewers working from a diff
alone reliably produce a few false positives — flagging parameterized queries as
injection, or missing middleware applied a layer up.

Say which findings you confirmed, which you believe are false positives and why,
and which you could not check. Then stop. Do not start fixing findings unless
the user asks.
