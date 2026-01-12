# Resolve Spec Folder

## Core Responsibility

Identify and validate the spec folder for UI design work.

## CRITICAL: Use Exact Folder Name

Before starting any work, you MUST resolve the spec folder path:

1. **Find the spec folder** by listing `agent-os/specs/`
2. **Use the EXACT folder name** - folders follow format `YYYY-MM-DD-HHMM-feature-name`
3. **NEVER create a new spec folder** - it already exists from earlier phases
4. **Store the resolved path** for consistent use throughout your work

## Workflow

### Step 1: Find Spec Folder

```bash
# List available specs
ls -la agent-os/specs/
```

If spec name argument provided:
- Match against folder names (partial match on feature name)
- Format: `YYYY-MM-DD-HHMM-feature-name`

If no argument:
- If single spec exists, use it
- If multiple specs exist, list them and ask user

### Step 2: Validate Spec Contents

```bash
# Verify spec.md exists
cat agent-os/specs/[this-spec]/spec.md

# Check for requirements
cat agent-os/specs/[this-spec]/planning/requirements.md 2>/dev/null

# Check for visual assets
ls -la agent-os/specs/[this-spec]/planning/visuals/ 2>/dev/null
```

**Required:**
- `spec.md` must exist and contain requirements

**Optional but useful:**
- `planning/requirements.md` - Additional requirements detail
- `planning/visuals/` - Visual mockups or screenshots for reference

### Step 3: Store Context

Remember for subsequent phases:
- Full spec folder path (e.g., `agent-os/specs/2025-01-12-1430-inventory-management/`)
- Feature name extracted from folder
- Whether visuals are available
- Any tech stack context from `agent-os/product/tech-stack.md`
