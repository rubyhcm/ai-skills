# GitNexus Setup & Usage Guide

**Repository:** https://github.com/abhigyanpatwari/GitNexus

GitNexus is a Code Intelligence Engine that builds a local knowledge graph of your codebase, providing "X-ray vision" into dependencies, execution flows, and impact analysis.

---

## Installation

### Prerequisites
- Node.js 18+ and npm
- Git repository

### Install Globally
```bash
npm install -g gitnexus
```

### Verify Installation
```bash
gitnexus --version
```

---

## Quick Start

### 1. Initialize in Your Repository
```bash
cd /path/to/your/project

# Initialize GitNexus
gitnexus init

# This creates .gitnexus/ directory with the knowledge graph database
```

### 2. Index Your Codebase
```bash
gitnexus analyze
```

GitNexus will analyze your code and build the dependency graph. This may take a few minutes for large codebases.

**IMPORTANT:** Always check if GitNexus is initialized before using:
```bash
# In scripts or AI agent workflows
if [ ! -d ".gitnexus" ]; then
  echo "GitNexus not initialized. Initializing..."
  gitnexus init
  gitnexus analyze
fi
```

### 3. Re-index After Changes
```bash
gitnexus analyze --incremental
```

Use `--incremental` to only re-index changed files.

---

## CLI Usage

### Query the Codebase
```bash
# Semantic search
gitnexus query "authentication logic"

# Find function definition
gitnexus query "func:CreateUser"

# Find all usages of a type
gitnexus query "type:UserRepository"
```

### Impact Analysis
```bash
# Analyze downstream impact (what depends on this?)
gitnexus impact --target "UserService" --direction downstream

# Analyze upstream dependencies (what does this depend on?)
gitnexus impact --target "UserRepository" --direction upstream
```

### Get Symbol Context
```bash
# 360-degree view of a symbol
gitnexus context --name "UserService"
```

### Detect Changes
```bash
# Map git diff to functional processes
gitnexus detect-changes
```

---

## MCP Integration (Model Context Protocol)

If your AI environment supports MCP, GitNexus provides tools for direct integration:

### Available MCP Tools

**query({query: "..."})**
- Semantic and graph-based codebase search
- Example: `query({query: "authentication middleware"})`

**impact({target: "...", direction: "upstream|downstream"})**
- Analyze blast radius before refactoring
- Example: `impact({target: "UserService.Create", direction: "downstream"})`

**context({name: "..."})**
- Get callers, callees, and functional cluster
- Example: `context({name: "UserRepository"})`

**detect_changes()**
- Map current git diffs to functional processes
- Example: `detect_changes()`

### Enable MCP in Your IDE

**Cursor IDE:**
Add to `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "gitnexus": {
      "command": "gitnexus",
      "args": ["mcp"]
    }
  }
}
```

**Kiro CLI:**
GitNexus MCP tools are auto-detected if `gitnexus` is in `$PATH`.

---

## Workflow Examples

### Before Refactoring a Shared Interface
```bash
# Check what will break
gitnexus impact --target "UserRepository" --direction downstream

# Review all implementations
gitnexus query "implements:UserRepository"
```

### Understanding a New Codebase
```bash
# Find entry points
gitnexus query "func:main"

# Explore authentication flow
gitnexus context --name "AuthMiddleware"
gitnexus query "calls:AuthMiddleware"
```

### Code Review Preparation
```bash
# See what changed functionally
gitnexus detect-changes

# Check impact of your changes
git diff --name-only | xargs -I {} gitnexus impact --target {}
```

---

## Configuration

### `.gitnexus/config.json`
```json
{
  "exclude": [
    "node_modules",
    "vendor",
    "dist",
    "build",
    ".git"
  ],
  "languages": ["go", "typescript", "python"],
  "indexDepth": 3
}
```

### Environment Variables
```bash
export GITNEXUS_DB_PATH=".gitnexus/db"
export GITNEXUS_LOG_LEVEL="info"
```

---

## Best Practices

1. **Initialize First** - Always check if `.gitnexus/` exists before using GitNexus commands
2. **Index Regularly** - Run `gitnexus analyze --incremental` after pulling changes
3. **Pre-Refactor Analysis** - Always run `impact` before modifying shared interfaces
4. **Commit Graph Updates** - Add `.gitnexus/` to `.gitignore` (graph is local)
5. **CI Integration** - Run `gitnexus analyze` in CI to validate graph consistency

---

## Troubleshooting

### Index is Stale
```bash
gitnexus analyze --force
```

### Large Codebase Performance
```bash
# Index only specific directories
gitnexus analyze --path ./internal --path ./pkg
```

### Clear and Rebuild
```bash
rm -rf .gitnexus
gitnexus init
gitnexus analyze
```

---

## Integration with AI Agents

AI agents MUST use GitNexus for:
- **Before refactoring** - Run `impact` to check blast radius
- **Understanding code** - Use `context` and `query` instead of grepping
- **Change validation** - Run `detect-changes` to verify functional correctness

### Initialization Check (MANDATORY)
Before using any GitNexus command, agents MUST verify initialization:
```bash
# Check if .gitnexus/ exists
if [ ! -d ".gitnexus" ]; then
  echo "[AGENT] GitNexus not initialized. Initializing..."
  gitnexus init
  gitnexus analyze
  echo "[AGENT] GitNexus initialization complete"
fi
```

### Agent Workflow
```bash
# 1. Ensure GitNexus is ready
[ -d .gitnexus ] || (gitnexus init && gitnexus analyze)

# 2. Before refactoring shared interface
gitnexus impact --target "UserRepository" --direction downstream

# 3. Review blast radius
gitnexus context --name "UserRepository"

# 4. Proceed with changes only if safe

# 5. After changes, verify functional impact
gitnexus detect-changes
```

See [ai-toolchain.md](../.kiro/steering/ai-toolchain.md) for enforcement rules.
