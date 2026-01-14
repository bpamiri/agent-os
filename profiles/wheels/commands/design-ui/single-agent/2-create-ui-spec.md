Now that you have the spec folder resolved, create the UI specification by following these instructions:

{{workflows/design/analyze-requirements}}

{{workflows/design/create-ui-mockups}}

{{workflows/design/create-data-contracts}}

## Display confirmation and next step

Display the following message to the user (replace `[this-spec]` with the actual folder name):

```
UI design complete!

Created in `agent-os/specs/[this-spec]/ui/`:
- ui-spec.md - Design decisions and overview
- pages/*.html - HTML mockups with DaisyUI
- components/*.html - Reusable component patterns
- data-contracts/contracts.md - Data requirements per view

Review the mockups to ensure they match your vision.

NEXT STEP: Run `/agent-os:create-tasks` to generate your tasks list, which will now include UI implementation tasks.
```

{{UNLESS standards_as_claude_code_skills}}
## User Standards & Preferences Compliance

IMPORTANT: Ensure that the UI design IS ALIGNED and DOES NOT CONFLICT with the user's preferences and standards as detailed in the following files:

{{standards/*}}
{{ENDUNLESS standards_as_claude_code_skills}}
