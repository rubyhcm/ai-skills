# Gemini CLI Instructions

This workspace follows strict Go Clean Architecture and agent tooling rules.
Detailed rules are found in the `.rules/` directory. Do not guess the architecture; always refer to `.rules/architecture.md` and `.rules/go-conventions.md`.

## Mandatory Agent Toolchain
You MUST follow the AI Toolchain rules defined below:

# AI Toolchain Rules

This project enforces the strict usage of **RTK (Rust Token Killer)**, **ICM (Interactive Context Management)**, and **GitNexus** across all AI agents, IDEs (Cursor, Kiro), and Assistants (Claude, Copilot). 

> **Important Setup Note:** RTK, ICM, and GitNexus are installed globally on the local host machine. They are available in the system `$PATH`. Do **NOT** attempt to install them, and do not expect them to be listed in project-level dependency files (e.g., `go.mod`, `package.json`). Assume they are always ready to use.

## 1. RTK & ICM (Token Efficiency & Output Management)
Repository: https://github.com/rtk-ai/rtk

**RTK** and **ICM** are required to filter noise, compress standard terminal outputs, and reduce token consumption. 
Whenever an AI agent executes a shell command, it MUST use RTK/ICM to wrap the command.

- **Shell Commands:** Always prefix standard information-gathering commands with `rtk` (unless a transparent shell hook is already active).
  - Use `rtk git status`, `rtk git diff`, `rtk git log` instead of raw git commands.
  - Use `rtk ls`, `rtk find` instead of standard `ls` or `find`.
  - Use `rtk test <command>` (e.g., `rtk test go test ./...`) so that the AI only receives failure traces and ignores passing noise.
  - Use `rtk read <file>` instead of `cat <file>` to view file contents with smart truncation.
- **ICM (Interactive Context Management):** Use `icm` commands when instructed or available to manage chat context, clear redundant history, and focus the context window on relevant code sections.

## 2. GitNexus (Code Intelligence & Impact Analysis)
Repository: https://github.com/abhigyanpatwari/GitNexus

**GitNexus** is our Code Intelligence Engine. It builds a local knowledge graph of the codebase to provide "X-ray vision" into dependencies and execution flows.
All AI agents must leverage GitNexus for repository understanding, refactoring, and impact analysis.

**CRITICAL: Initialize GitNexus Before First Use**
```bash
# Check if .gitnexus/ exists
if [ ! -d ".gitnexus" ]; then
  gitnexus init
  gitnexus analyze
fi

# Re-index after pulling changes
gitnexus analyze --incremental
```

- **MCP Tools:** If your environment supports MCP (Model Context Protocol), use GitNexus MCP tools:
  - `query({query: "..."})` for semantic and graph-based codebase search.
  - `impact({target: "...", direction: "upstream|downstream"})` before modifying any shared interface or critical struct. This analyzes the blast radius of your changes.
  - `context({name: "..."})` to get a 360-degree view of a symbol (callers, callees, functional cluster).
  - `detect_changes()` to map current git diffs to functional processes.
- **CLI Fallback:** If MCP is unavailable, use the equivalent GitNexus CLI commands to perform graph queries and impact analysis *before* making cross-cutting codebase changes.

## 3. Git Operations & Autonomy Constraints
AI Agents MUST NOT autonomously stage or commit changes to version control without explicit, granular user permission.
- **STRICT FORBIDDEN:** Do NOT execute ANY `git add` command (even for a single specific file) unless the user has explicitly told you to "stage this file", "git add this", or "prepare a commit with these changes".
- **STRICT FORBIDDEN:** Do NOT use wildcard staging like `git add .` or `git add -A` under any circumstances.
- **STRICT FORBIDDEN:** Do NOT execute `git commit` unless the user explicitly asks you to "commit the changes".
- **REQUIRED ACTION:** If you have finished a task and the user hasn't mentioned git, simply stop and report that the files are modified in the working directory. Do NOT attempt to stage them "just in case".

## Enforcement Checklist for Agents
1. Did I check if `.gitnexus/` exists? If not, did I run `gitnexus init && gitnexus analyze`?
2. Did I wrap my test execution or git commands with `rtk`?
3. Did I use GitNexus (`impact` or `context`) to verify the blast radius before refactoring a core domain struct or service interface?
4. Am I optimizing my context window using ICM principles rather than dumping entire files?
5. Did I refrain from running ANY `git add` or `git commit` command? (I must only do this if the user explicitly said to stage/commit).

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **ai-backend-go** (824 symbols, 889 relationships, 1 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/ai-backend-go/context` | Codebase overview, check index freshness |
| `gitnexus://repo/ai-backend-go/clusters` | All functional areas |
| `gitnexus://repo/ai-backend-go/processes` | All execution flows |
| `gitnexus://repo/ai-backend-go/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
