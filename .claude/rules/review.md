# Review checklist (ai-council-plugin)

This repo is only the Claude Code integration; the engine is the `@mugzie/ai-council` npm package. Check every diff against these.

1. Any change users would notice bumps `version` in both `.claude-plugin/plugin.json` and the plugin entry in `.claude-plugin/marketplace.json`, to the same value.
2. Each command's `allowed-tools` lists both spellings of its MCP tool (`mcp__plugin_ai-council_ai-council__<tool>` and `mcp__plugin:ai-council:ai-council__<tool>`), and `<tool>` is one the server exposes.
3. Argument tables in `commands/*.md` and `skills/council-review/SKILL.md` map to real tool parameters, and the incompatible combinations (`pr` with `gitBranch`/`gitCommit`/`gitRange`/`reviewBranch`) stay documented.
4. No hook is added: a `Stop` or `PostToolUse` hook would run a paid review every turn on the user's keys.
5. No key, licence or token is stored in the plugin; keys come from the shell environment only.
6. `.mcp.json` launches the published package with `npx -y`; no local path or unpinned git source.
7. README's command, skill and environment-variable tables match `commands/`, `skills/` and what the engine reads.
8. Commands report the verdict, confidence, dissent and findings, and do not act on findings unless the user asks.
