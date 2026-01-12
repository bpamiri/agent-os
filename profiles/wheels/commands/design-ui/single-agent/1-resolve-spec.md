The FIRST STEP is to resolve and confirm the spec folder for UI design.

You need the spec folder containing:
- `agent-os/specs/[this-spec]/spec.md` with feature requirements

## Finding the Spec

1. **If spec name was provided with command** (e.g., `/design-ui inventory-management`):
   - Search for matching folder in `agent-os/specs/`
   - Use partial matching on the feature name portion (folders use format `YYYY-MM-DD-HHMM-feature-name`)

2. **If no spec name provided**:
   - List available specs: `ls agent-os/specs/`
   - If only one spec exists, use that one
   - If multiple specs exist, ask user which one to design

3. **Validate the spec folder has spec.md**:
   - Read `agent-os/specs/[this-spec]/spec.md`
   - If not found, output error and ask user to run `/write-spec` first

## Error Handling

If you cannot find a spec folder or spec.md, output:

```
I couldn't find a spec to design UI for.

Please either:
1. Provide the spec name: `/design-ui [feature-name]`
2. Run `/write-spec` first to create your spec.md

Available specs:
[list any found in agent-os/specs/]
```

{{UNLESS compiled_single_command}}
## Display confirmation and next step

Once you've confirmed the spec folder exists and contains spec.md, output the following message (replace `[this-spec]` with the actual folder name):

```
Spec resolved: `agent-os/specs/[this-spec]/`

NEXT STEP: Run the command, 2-create-ui-spec.md
```
{{ENDUNLESS compiled_single_command}}
