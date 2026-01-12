# Analyze Requirements for UI Design

## Core Responsibility

Extract UI-relevant information from the spec to inform mockup creation.

## Workflow

### Step 1: Read Spec Documents

```bash
cat agent-os/specs/[this-spec]/spec.md
cat agent-os/specs/[this-spec]/planning/requirements.md 2>/dev/null
```

### Step 2: Extract UI Requirements

From the spec, identify:

1. **User Stories** - What views do users need?
2. **Data Entities** - What information is displayed/edited?
3. **Actions** - What can users do? (CRUD, workflows, etc.)
4. **Constraints** - Mobile requirements, accessibility, performance

### Step 3: Create Mental Map

Build understanding of the feature structure:

```
Feature: [Name]
├── Views Needed
│   ├── List/Index (browse collections)
│   ├── Detail/Show (view single item)
│   ├── Form (create/edit)
│   └── [Special views: dashboard, report, wizard]
├── Components Needed
│   ├── Data display (tables, cards, stats)
│   ├── Navigation (tabs, breadcrumbs, pagination)
│   ├── Forms (inputs, selects, validation)
│   └── Feedback (alerts, toasts, modals)
└── Interactions
    ├── HTMX patterns (if applicable)
    ├── Modal dialogs
    └── Inline editing
```

### Step 4: Determine View List

Map requirements to views:

| Requirement Type | View Needed |
|-----------------|-------------|
| "List of X" / "Browse X" | Index view (`pages/index.html`) |
| "View X details" | Show view (`pages/show.html`) |
| "Create new X" | New form (`pages/new.html`) |
| "Edit X" | Edit form (`pages/edit.html`) |
| "Dashboard" / "Overview" | Dashboard (`pages/dashboard.html`) |
| "Quick action" / "Confirm" | Modal (`components/_modal.html`) |

### Step 5: Document Analysis

Create mental map of:
- Views to create
- Components to extract
- Data each view needs
- Actions available per view
