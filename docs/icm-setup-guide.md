# ICM (Interactive Context Management) Setup & Usage Guide

**Repository:** https://github.com/rtk-ai/icm

ICM is a context window optimizer for AI conversations. It helps manage chat history, clear redundant information, and focus the context on relevant code sections.

---

## Installation

### Prerequisites
- Python 3.8+
- pip

### Install Globally
```bash
pip install icm-ai
```

Or install from source:
```bash
git clone https://github.com/rtk-ai/icm.git
cd icm
pip install -e .
```

### Verify Installation
```bash
icm --version
```

---

## Quick Start

### 1. Initialize in Your Project
```bash
cd /path/to/your/project
icm init
```

This creates `.icm/` directory with context management configuration.

### 2. Start Context Session
```bash
icm start
```

ICM will monitor your AI conversation and suggest context optimizations.

---

## Core Commands

### Context Management

**Clear Redundant History**
```bash
icm clear
```
Removes duplicate information and resolved issues from context.

**Focus on Specific Files**
```bash
icm focus internal/service/user_service.go internal/domain/user.go
```
Keeps only specified files in context, removes everything else.

**Snapshot Current Context**
```bash
icm snapshot "before-refactoring-auth"
```
Saves current context state for later restoration.

**Restore Snapshot**
```bash
icm restore "before-refactoring-auth"
```

**List Snapshots**
```bash
icm snapshots
```

---

## Context Analysis

### View Context Usage
```bash
icm status
```
Shows:
- Current token count
- Context window utilization
- Top files by token consumption
- Redundant entries

### Analyze Context Bloat
```bash
icm analyze
```
Identifies:
- Duplicate code blocks
- Stale conversation threads
- Unused file references
- Optimization opportunities

---

## Advanced Usage

### Auto-Cleanup Mode
```bash
icm auto --threshold 80
```
Automatically clears context when utilization exceeds 80%.

### Selective Retention
```bash
# Keep only errors and warnings
icm filter --level error,warn

# Keep only recent messages (last 10)
icm trim --last 10

# Keep only specific topics
icm filter --topic "authentication,database"
```

### Context Compression
```bash
# Compress old messages (summarize)
icm compress --older-than 1h

# Compress everything except recent
icm compress --keep-recent 5
```

---

## Integration with AI Workflows

### Before Starting a New Feature
```bash
icm clear
icm focus internal/service/ internal/domain/
```

### During Long Debugging Sessions
```bash
# Every 30 minutes
icm analyze
icm clear
```

### Before Major Refactoring
```bash
icm snapshot "pre-refactor-$(date +%Y%m%d-%H%M%S)"
icm focus <files-to-refactor>
```

### After Completing a Task
```bash
icm clear
icm status
```

---

## Configuration

### `.icm/config.yaml`
```yaml
auto_clear:
  enabled: true
  threshold: 85  # Clear when context > 85%

retention:
  max_messages: 50
  max_age_hours: 24

compression:
  enabled: true
  min_age_minutes: 30

focus:
  default_paths:
    - internal/
    - pkg/
  exclude_patterns:
    - "*_test.go"
    - "*.pb.go"
    - "mock_*.go"
```

### Environment Variables
```bash
export ICM_AUTO_CLEAR=true
export ICM_THRESHOLD=80
export ICM_LOG_LEVEL=info
```

---

## Best Practices

1. **Clear Regularly** - Run `icm clear` after completing each task
2. **Focus Early** - Use `icm focus` at the start of each feature
3. **Snapshot Before Refactoring** - Always create snapshots before major changes
4. **Monitor Status** - Check `icm status` periodically during long sessions
5. **Auto Mode for Long Sessions** - Enable `icm auto` for extended debugging

---

## Workflow Examples

### Starting a New Feature
```bash
icm clear
icm snapshot "baseline-$(date +%Y%m%d)"
icm focus internal/service/user_service.go internal/domain/user.go
```

### Mid-Session Cleanup
```bash
icm analyze
# Review suggestions
icm clear
icm status
```

### Context Too Large
```bash
icm status
# If > 90%:
icm compress --older-than 30m
icm trim --last 20
icm focus <critical-files>
```

### Switching Tasks
```bash
icm snapshot "task-A-paused"
icm clear
icm focus <task-B-files>
# Later:
icm restore "task-A-paused"
```

---

## Integration with AI Agents

AI agents should use ICM to:
- **Prevent context bloat** - Clear after each completed task
- **Optimize token usage** - Focus on relevant files only
- **Maintain conversation quality** - Remove redundant history
- **Enable long sessions** - Use compression and auto-cleanup

### Agent Commands
```bash
# Before starting work
icm clear && icm focus <relevant-files>

# During work (every 10-15 messages)
icm analyze && icm clear

# After completing task
icm clear && icm status
```

---

## Troubleshooting

### Context Not Clearing
```bash
icm clear --force
```

### Snapshots Not Restoring
```bash
icm snapshots --verify
icm restore <snapshot-name> --force
```

### High Token Usage Despite Clearing
```bash
icm analyze --verbose
icm compress --aggressive
icm focus <minimal-file-set>
```

### Reset ICM State
```bash
rm -rf .icm
icm init
```

---

## IDE Integration

### Cursor IDE
Add to workspace settings:
```json
{
  "icm.autoClean": true,
  "icm.threshold": 80,
  "icm.focusPaths": ["internal/", "pkg/"]
}
```

### Kiro CLI
ICM is auto-detected if installed globally. Use `/context` commands in Kiro to trigger ICM operations.

---

## Performance Tips

1. **Use Focus Aggressively** - Only keep 3-5 files in context at a time
2. **Compress Old Messages** - Messages older than 1 hour can be summarized
3. **Clear After Each Task** - Don't carry context between unrelated tasks
4. **Monitor Token Count** - Keep context below 80% of window size
5. **Snapshot Frequently** - Cheap operation, enables quick rollback

---

## See Also

- [RTK Setup Guide](rtk-setup-guide.md) - Token optimization for command outputs
- [AI Toolchain Rules](../.kiro/steering/ai-toolchain.md) - Enforcement policies
