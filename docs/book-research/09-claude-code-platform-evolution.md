# Claude Code Platform Evolution

## Tracking the Platform (Issue #215)

ArcKit actively tracks Claude Code releases for capabilities that improve the plugin. Issue #215 consolidates tracking from v2.1.83 through v2.1.107.

## High-Value Capabilities Identified (19 Items)

| # | Capability | Claude Code Version | Impact on ArcKit |
|---|-----------|-------------------|-----------------|
| 1 | Plugin `userConfig` | v2.1.83 | API keys and org config in plugin.json |
| 2 | Conditional `if` on hooks | v2.1.85 | Reduce process spawning; compound-command bug fixed in v2.1.89 |
| 3 | Agent `initialPrompt` | v2.1.83 | Auto-submit first turn for 9 agents |
| 4 | Skills `paths:` globs | v2.1.84 | Scope 4 skills to relevant file patterns |
| 5 | `FileChanged`/`CwdChanged` hooks | v2.1.83 | Reactive context injection |
| 6 | `TaskCreated` hook | v2.1.84 | Agent tracking with blocking behavior (v2.1.89) |
| 7 | `PostCompact` hook | v2.1.76 | Re-inject context after compaction |
| 8 | `${CLAUDE_PLUGIN_DATA}` | v2.1.78 | Persistent state for session learning |
| 9 | MCP `headersHelper` env vars | v2.1.85 | Shared auth for multiple MCP servers |
| 10 | `PermissionDenied` hook | v2.1.89 | Handle auto-mode denials with retry |
| 11 | `defer` permission for PreToolUse | v2.1.89 | Headless/CI pause-and-resume |
| 12 | Skill description 250-char cap | v2.1.86 | **Raised to 1,536 chars in v2.1.105** -- can restore richer descriptions |
| 13 | `keep-coding-instructions` frontmatter | v2.1.94 | Persist static instructions across compaction |
| 14 | `hookSpecificOutput.sessionTitle` | v2.1.94 | Session-aware learning in session-learner.mjs |
| 15 | `monitors` top-level manifest key | v2.1.105 | Background monitors for artifact watch and stale-doc detection |
| 16 | Skill description cap 250→1,536 | v2.1.105 | Re-evaluate 4 skill descriptions for better discoverability |
| 17 | PreCompact hook blocking | v2.1.105 | Block compaction mid-session when critical state is still needed |
| 18 | `Monitor` tool | v2.1.98 | Stream events from long-running background scripts (research agents, govreposcrape) |
| 19 | `/claude-api` Managed Agents coverage | v2.1.98 | Aligns with arckit-research managed agent deployment (PR #282) |

## Platform Fixes That Affected ArcKit

### v2.1.89
- Hook `if` compound-command bug fix
- `file_path` now absolute in PreToolUse/PostToolUse hooks
- Hook output >50K saved to disk instead of context
- MCP non-blocking startup + 5s connection bound
- Autocompact thrash loop fix

### v2.1.90
- PostToolUse format-on-save hook no longer causes "File content has changed" failures
- PreToolUse JSON exit-code-2 blocking fix (affects ArcKit's blocking hooks)
- MCP tool schema JSON.stringify eliminated (performance win for 5 MCP servers)
- SSE transport now linear-time (was quadratic)
- `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` for offline environments

### v2.1.91
- Plugin `bin/` executables -- ArcKit's 6 bash scripts could ship under `bin/`
- `disableSkillShellExecution` setting -- could break ArcKit commands that invoke bash scripts

### v2.1.92
- Stop hook semantics fix -- restores `preventContinuation:true` behavior (relevant to session-learner.mjs)
- Plugin MCP servers stuck 'connecting' fix (affects ArcKit's 5 MCP servers)
- Subagent spawning fix in tmux (affects ArcKit's 10 agents)
- Write tool 60% faster for large files with tabs/`&`/`$` (common in architecture documents)

### v2.1.94
- `keep-coding-instructions` frontmatter -- persist instructions across compaction
- Default effort changed medium->high for API-key users
- Fixed `${CLAUDE_PLUGIN_ROOT}` resolving for local-marketplace plugins
- Fixed agents stuck after 429 with long Retry-After (affects 10 ArcKit agents)

### v2.1.97 (Current Minimum)
- MCP SSE memory leak ~50 MB/hr on reconnects fixed (benefits 5 MCP servers)
- 429 exponential backoff fix (was burning all retries in ~13s, benefits research agents)
- Stop/SubagentStop hooks no longer fail on long sessions (benefits session-learner.mjs)
- Subagent worktree/cwd leak fixed (benefits 10 agents)
- `claude plugin update` now detects new remote commits (critical for ArcKit distribution)
- Compaction dedup of multi-MB subagent transcript files
- Session transcript size reduced by skipping empty hook entries (benefits 18 hooks)

### v2.1.98
- Subagent MCP tool inheritance from dynamically-injected servers fixed (affects 10 agents × 5 MCP servers)
- Compound Bash permission bypass security fix
- `Monitor` tool added for streaming events from background scripts
- `/claude-api` skill now covers Managed Agents alongside Claude API
- Bash command injection in POSIX `which` fallback fixed

### v2.1.101
- Sub-agents in isolated worktrees can now Read/Edit their own worktree
- MCP tools now available on first turn of headless/remote-trigger sessions
- `permissions.deny` now overrides PreToolUse hook `ask` (safer default)
- Plugin slash commands resolving to wrong plugin with duplicate `name:` fixed
- Skills now honor `context: fork` and `agent` frontmatter fields
- OS CA certificate store trusted by default (enterprise TLS proxies work out-of-box)

### v2.1.105 (Current Recommended)
- `monitors` top-level plugin manifest key for background monitors
- Skill description cap raised from 250 to 1,536 characters
- PreCompact hook can now block compaction (exit 2 or `{"decision":"block"}`)
- Marketplace plugins with `package.json` + lockfile auto-install deps (critical for Paperclip TS plugin)
- Marketplace auto-update no longer leaves broken state on file-lock during update
- Stalled API streams abort after 5 min with non-streaming retry
- WebFetch strips `<style>`/`<script>` tags (benefits research agents)
- Stale agent worktree cleanup for squash-merged PRs

### v2.1.107
- Thinking hints shown sooner during long operations (no ArcKit impact)

## Minimum Version History

| Date | Min Version | Reason |
|------|------------|--------|
| Pre-April 2026 | v2.1.90 | PreToolUse blocking fix, MCP performance |
| 9 April 2026 | v2.1.97 | `claude plugin update` fix, MCP memory leak, 429 backoff |
| 14 April 2026 | v2.1.97 (min), v2.1.105 (recommended) | Marketplace plugin deps auto-install, `monitors` manifest, skill description cap raised |

## The Relationship Between ArcKit and Claude Code

ArcKit is one of the most complex Claude Code plugins in existence, pushing the platform's capabilities in areas like:
- Hook system (17 registered handlers across 7 event types)
- Agent system (10 autonomous agents with isolated contexts)
- MCP integration (5 external servers)
- Multi-format distribution (7 formats from one source)

This makes ArcKit both a beneficiary and a stress-tester of Claude Code features. Bugs discovered through ArcKit usage have contributed to platform improvements.
