# RTK (Rust Token Killer) Setup & Usage Guide

**Repository:** https://github.com/rtk-ai/rtk

RTK is a token efficiency tool that filters noise, compresses standard terminal outputs, and reduces token consumption in AI conversations. It wraps common shell commands to provide only relevant information.

---

## Installation

### Prerequisites
- Rust 1.70+ and Cargo (for building from source)
- Or use pre-built binaries

### Install via Cargo
```bash
cargo install rtk-ai
```

### Install from Pre-built Binary
```bash
# Linux/macOS
curl -fsSL https://rtk.ai/install.sh | sh

# Or download from releases
wget https://github.com/rtk-ai/rtk/releases/latest/download/rtk-linux-x64
chmod +x rtk-linux-x64
sudo mv rtk-linux-x64 /usr/local/bin/rtk
```

### Install from Source
```bash
git clone https://github.com/rtk-ai/rtk.git
cd rtk
cargo build --release
sudo cp target/release/rtk /usr/local/bin/
```

### Verify Installation
```bash
rtk --version
```

---

## Quick Start

### Basic Usage Pattern
```bash
# Instead of:
git status

# Use:
rtk git status
```

RTK filters out noise and presents only actionable information.

---

## Core Commands

### Git Operations

**Status (Compressed)**
```bash
rtk git status
```
Shows only:
- Modified files
- Untracked files
- Staged changes
(Removes verbose git hints and formatting)

**Diff (Smart Truncation)**
```bash
rtk git diff
```
Shows:
- Changed lines with context
- Truncates large diffs
- Highlights important changes

**Log (Condensed)**
```bash
rtk git log
```
Shows:
- Commit hash (short)
- Author
- Date (relative)
- Message (first line only)

**Branch List**
```bash
rtk git branch
```
Shows only branch names, removes decorations.

---

### File Operations

**List Files (Filtered)**
```bash
rtk ls
```
Removes:
- Hidden files (unless `-a`)
- Metadata columns
- Color codes

**Find Files (Relevant Only)**
```bash
rtk find . -name "*.go"
```
Shows only file paths, removes permission errors and warnings.

**Read File (Smart Truncation)**
```bash
rtk read internal/service/user_service.go
```
Shows:
- First 100 lines by default
- Truncation indicator if file is longer
- Option to show more: `rtk read <file> --lines 500`

---

### Test Execution (Failure-Only Mode)

**Run Tests (Show Only Failures)**
```bash
rtk test go test ./...
```
Output:
- ✅ Only failed test names and error messages
- ❌ Suppresses passing test noise
- 📊 Summary: X passed, Y failed

**Run with Coverage**
```bash
rtk test go test -cover ./...
```
Shows only coverage percentage, removes verbose output.

**Run Specific Test**
```bash
rtk test go test -run TestUserService_Create ./internal/service/
```

---

### Build & Compile (Error-Only Mode)

**Build (Show Only Errors)**
```bash
rtk build go build ./cmd/api
```
Shows:
- Compilation errors
- Warnings (optional, use `--no-warnings` to hide)
- Suppresses successful build messages

**Lint (Violations Only)**
```bash
rtk lint golangci-lint run ./...
```
Shows only linting violations, removes progress bars and stats.

---

## Advanced Usage

### Custom Filters

**Create Filter Profile**
```bash
rtk config add-filter "test" --show-failures --hide-passing
rtk config add-filter "git" --compact --no-color
```

**Use Filter Profile**
```bash
rtk --profile test go test ./...
```

### Transparent Shell Hook (Auto-RTK)

**Enable for Current Session**
```bash
eval "$(rtk shell-hook)"
```

Now all commands are automatically wrapped:
```bash
git status  # Actually runs: rtk git status
go test ./... # Actually runs: rtk test go test ./...
```

**Enable Permanently**
Add to `~/.bashrc` or `~/.zshrc`:
```bash
eval "$(rtk shell-hook)"
```

**Disable Hook**
```bash
rtk shell-hook --disable
```

---

## Configuration

### `.rtk/config.toml`
```toml
[filters]
git_status = { compact = true, no_hints = true }
git_diff = { max_lines = 200, context = 3 }
git_log = { max_commits = 20, short_hash = true }

[test]
show_passing = false
show_failures = true
show_summary = true

[build]
show_warnings = true
show_errors = true
show_success = false

[file]
max_lines = 100
truncate_indicator = "... (truncated, use --lines N to see more)"
```

### Environment Variables
```bash
export RTK_MAX_OUTPUT_LINES=500
export RTK_COMPACT_MODE=true
export RTK_COLOR=false
```

---

## Best Practices

1. **Always Use RTK for AI Interactions** - Wrap all shell commands with `rtk`
2. **Enable Shell Hook for Convenience** - Auto-wrap all commands
3. **Use Test Mode for CI** - `rtk test` shows only failures
4. **Read Large Files Carefully** - Use `rtk read` instead of `cat`
5. **Compress Git Logs** - `rtk git log` for commit history

---

## Workflow Examples

### Code Review Preparation
```bash
rtk git status
rtk git diff
rtk git log --since="1 week ago"
```

### Running Tests in AI Session
```bash
rtk test go test ./...
# Only failures shown, AI can focus on fixing them
```

### Exploring Codebase
```bash
rtk find . -name "*_service.go"
rtk read internal/service/user_service.go
```

### Build Verification
```bash
rtk build go build ./cmd/api
rtk lint golangci-lint run ./...
```

---

## Integration with AI Agents

AI agents MUST use RTK for:
- **All git commands** - `rtk git status`, `rtk git diff`, `rtk git log`
- **Test execution** - `rtk test <test-command>`
- **File reading** - `rtk read <file>` instead of `cat`
- **Build/compile** - `rtk build <build-command>`
- **Linting** - `rtk lint <lint-command>`

### Agent Command Pattern
```bash
# ❌ WRONG (verbose, wastes tokens)
git status
cat internal/service/user_service.go
go test ./...

# ✅ CORRECT (compressed, token-efficient)
rtk git status
rtk read internal/service/user_service.go
rtk test go test ./...
```

---

## Comparison: Before vs After RTK

### Git Status
**Before (142 tokens):**
```
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   internal/service/user_service.go
        modified:   internal/domain/user.go

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        internal/service/user_service_test.go

no changes added to commit (use "git add" and/or "git commit -a")
```

**After (18 tokens):**
```
Modified:
  internal/service/user_service.go
  internal/domain/user.go
Untracked:
  internal/service/user_service_test.go
```

**Token Savings: 87%**

---

### Test Output
**Before (523 tokens):**
```
=== RUN   TestUserService_Create
=== RUN   TestUserService_Create/success
=== RUN   TestUserService_Create/invalid_email
=== RUN   TestUserService_Create/duplicate_email
--- PASS: TestUserService_Create (0.00s)
    --- PASS: TestUserService_Create/success (0.00s)
    --- PASS: TestUserService_Create/invalid_email (0.00s)
    --- FAIL: TestUserService_Create/duplicate_email (0.00s)
        user_service_test.go:45: Expected error ErrAlreadyExists, got nil
=== RUN   TestUserService_GetByID
=== RUN   TestUserService_GetByID/success
=== RUN   TestUserService_GetByID/not_found
--- PASS: TestUserService_GetByID (0.00s)
    --- PASS: TestUserService_GetByID/success (0.00s)
    --- PASS: TestUserService_GetByID/not_found (0.00s)
FAIL
FAIL    internal/service    0.123s
```

**After (42 tokens):**
```
❌ TestUserService_Create/duplicate_email
   user_service_test.go:45: Expected error ErrAlreadyExists, got nil

Summary: 4 passed, 1 failed
```

**Token Savings: 92%**

---

## Troubleshooting

### RTK Not Found
```bash
which rtk
# If empty, add to PATH:
export PATH="$PATH:/usr/local/bin"
```

### Shell Hook Not Working
```bash
# Re-enable
eval "$(rtk shell-hook)"

# Verify
type git
# Should show: git is a function
```

### Output Still Verbose
```bash
# Force compact mode
rtk --compact git status

# Or update config
rtk config set compact true
```

### Disable RTK Temporarily
```bash
# Use raw command
command git status

# Or disable hook
rtk shell-hook --disable
```

---

## Performance Impact

RTK adds minimal overhead:
- **Latency:** < 5ms per command
- **Memory:** < 2MB resident
- **CPU:** Negligible (filtering is fast)

---

## See Also

- [ICM Setup Guide](icm-setup-guide.md) - Context window optimization
- [GitNexus Setup Guide](gitnexus-setup-guide.md) - Code intelligence
- [AI Toolchain Rules](../.kiro/steering/ai-toolchain.md) - Enforcement policies
