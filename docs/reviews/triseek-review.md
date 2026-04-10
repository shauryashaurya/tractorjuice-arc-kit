# TriSeek Review: Relevance to ArcKit

**Repository**: [Sagart-cactus/TriSeek](https://github.com/Sagart-cactus/TriSeek)
**Reviewed**: 2026-04-10
**Reviewer**: Claude Code (automated review)

## What is TriSeek?

TriSeek is a Rust-based local code search CLI that augments ripgrep with trigram-based indexing for
faster repeated searches on medium to large codebases (5K-500K+ files). It provides:

- **Immediate search** without indexing (`triseek "AuthConfig" .`)
- **Optional trigram index** for 12-20x speedups on repeated queries
- **Adaptive routing** between indexed search, direct scan, and ripgrep fallback
- **MCP server** (`triseek mcp serve`) exposing 5 tools for AI coding assistants
- **MIT licensed**, Rust workspace (~3,500 LOC), v0.2.1

## MCP Server Capabilities

TriSeek ships an MCP server (stdio, JSON-RPC 2.0) with these tools:

| Tool | Purpose |
|------|---------|
| `find_files` | Path/filename substring search |
| `search_content` | Literal or regex content search with line/column/preview |
| `search_path_and_content` | Two-stage: path glob + content query |
| `index_status` | Index health, file count, repo category |
| `reindex` | Incremental or full index rebuild |

Output is token-efficient: 20-item default limit, 100 max, 200-char preview truncation, deduplication.

## Relevance Assessment

### Not Directly Useful for ArcKit Core (Low Priority)

1. **ArcKit is not a code search tool** - ArcKit generates architecture governance artifacts
   (requirements, business cases, ADRs, stakeholder analyses). It doesn't need to search
   user source code to produce these documents.

2. **ArcKit's existing search is artifact-focused** - The `/arckit:search` command
   (`search-scan.mjs`) indexes ArcKit project artifacts (ARC-*.md files) by metadata
   (title, type, status, owner), not source code. TriSeek solves a different problem.

3. **ArcKit already has 5 MCP servers** for its domain:
   - AWS Knowledge (architecture docs)
   - Microsoft Learn (Azure docs)
   - Google Developer Knowledge (GCP docs)
   - Data Commons (data discovery)
   - govreposcrape (UK government repository search)

   These serve ArcKit's architecture governance mission. Adding a code search MCP would
   be out of scope.

4. **No code analysis commands exist** - ArcKit commands generate documents from templates
   and web research, not from scanning source code.

### Potentially Interesting Ideas (Future Consideration)

1. **Code-to-Architecture traceability** - A future `/arckit:code-alignment` command could
   use TriSeek's MCP to verify that codebase patterns match architecture decisions (e.g.,
   "ADR-003 says use PostgreSQL - does the code actually import pg?"). This would bridge
   the gap between governance artifacts and implementation reality. However, this is a
   speculative future direction, not a current need.

2. **Reuse assessment enhancement** - The existing `/arckit:gov-reuse` and
   `/arckit:gov-code-search` commands search *remote* government repositories via
   govreposcrape. TriSeek could complement this by searching the *local* codebase to
   identify what's already been built before recommending external reuse. Again, speculative.

3. **MCP server pattern reference** - TriSeek's MCP implementation (stdio, token-efficient
   output caps, error codes with retryability hints) is a clean reference for how to build
   well-behaved MCP servers. The output discipline (20-item default, 200-char preview,
   dedup) is worth noting if ArcKit ever builds custom MCP servers.

4. **Adaptive routing pattern** - TriSeek's three-strategy routing (indexed/direct-scan/
   ripgrep-fallback) with automatic selection based on repo size and query shape is an
   elegant pattern. ArcKit's search-scan.mjs could potentially adopt a similar tiered
   approach for artifact search at scale, though current project sizes don't warrant this.

### Not Applicable

- **Performance claims** (12-20x over ripgrep) are irrelevant since ArcKit doesn't perform
  code search at scale
- **Trigram indexing** is a code search technique, not useful for Markdown artifact search
- **Rust implementation** - ArcKit is Python CLI + JavaScript hooks + Markdown templates;
  no Rust in the stack
- **Installation/daemon model** - Adds deployment complexity for no clear ArcKit benefit

## Verdict

**No immediate action recommended.** TriSeek is a well-engineered code search tool, but it
solves a problem ArcKit doesn't have. ArcKit's value proposition is architecture governance
artifact generation, not source code analysis.

**Bookmark for future reference** if ArcKit ever adds code-to-architecture traceability
commands (verifying implementations match architecture decisions). The MCP server design
patterns are also a useful reference.

## Key Facts

| Attribute | Value |
|-----------|-------|
| Repository | Sagart-cactus/TriSeek |
| Language | Rust (96.8%) |
| License | MIT |
| Version | 0.2.1 |
| Stars | 1 |
| MCP Protocol | 2025-06-18 (stdio) |
| MCP Tools | 5 (find_files, search_content, search_path_and_content, index_status, reindex) |
| Crates | 6 (search-core, search-frecency, search-index, search-cli, search-bench, search-server) |
| Binary size | Single binary, prebuilt for macOS/Linux/Windows |
| Index format | Custom mmap-friendly binary (fast.idx) + bincode snapshots |
