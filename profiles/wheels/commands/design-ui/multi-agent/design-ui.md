# UI Design Process

You are creating user interface specifications for a feature spec. This command creates HTML mockups with DaisyUI components and data contracts that bridge the gap between your spec.md and implementation.

## PHASE 1: Resolve Spec Folder

First, identify which spec folder to design UI for:

1. **If spec name provided with command** (e.g., `/design-ui inventory-management`):
   - Search for matching folder in `agent-os/specs/`
   - Use partial matching on the feature name portion

2. **If no spec name provided**:
   - List available specs: `ls agent-os/specs/`
   - If only one spec exists, use that one
   - If multiple specs exist, ask user which one to design

3. **Validate** the spec contains `spec.md` with requirements

If you cannot find a spec folder, output:

```
I couldn't find a spec to design UI for.

Please either:
1. Provide the spec name: `/design-ui [feature-name]`
2. Run `/write-spec` first to create your spec.md
```

## PHASE 2: Create UI Specification

Use the **ui-designer** subagent to create the UI specification.

Provide the ui-designer with:
- The spec folder path: `agent-os/specs/[this-spec]/`
- The requirements from `spec.md`
- Any visual assets in `planning/visuals/` (if present)
- The project's tech stack context (from `agent-os/product/tech-stack.md` if exists)

The ui-designer will create the `ui/` folder structure inside the spec folder with:
- `ui-spec.md` - Design decisions and view overview
- `pages/` - HTML mockups for each view
- `components/` - Reusable UI component patterns
- `data-contracts/contracts.md` - Variable specifications per view

## PHASE 3: Inform User

Once the ui-designer has completed the UI specification, output:

```
UI design ready!

Created: `agent-os/specs/[this-spec]/ui/`

Contents:
- ui-spec.md - Design direction and view specifications
- pages/*.html - Framework-agnostic HTML mockups
- components/*.html - Reusable component patterns
- data-contracts/contracts.md - Data requirements per view

Review the mockups to ensure they match your vision.

NEXT STEP: Run `/create-tasks` to generate implementation tasks.
```
