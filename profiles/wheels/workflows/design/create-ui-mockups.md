# Create UI Mockups

## Core Responsibility

Generate HTML mockups using DaisyUI components with framework-agnostic markers.

## Workflow

### Step 1: Get Design Direction

**ALWAYS invoke the frontend-design skill first** for creative direction:

```
Skill: frontend-design

I'm designing [describe the feature] for a [type of application].

Context:
- Purpose: [What problem does this solve?]
- Users: [Who will use this?]
- Tone: [Professional, playful, minimal, etc.]

Constraints:
- Framework: DaisyUI v5 + Tailwind CSS
- Output: Static HTML (for server-side templating)
- Theme: [Specify if known, otherwise ask for recommendation]

Please provide:
1. Aesthetic direction
2. Color/typography recommendations
3. Layout approach
4. Key differentiating elements
```

### Step 2: Create Directory Structure

```bash
mkdir -p agent-os/specs/[this-spec]/ui/pages
mkdir -p agent-os/specs/[this-spec]/ui/components
mkdir -p agent-os/specs/[this-spec]/ui/data-contracts
```

### Step 3: Generate ui-spec.md

Write `agent-os/specs/[this-spec]/ui/ui-spec.md` documenting:

- Design Direction (from frontend-design skill)
- Views Overview table
- View Specifications for each view
- Reusable Components list
- Accessibility Notes
- Implementation Guidance

### Step 4: Generate Page Mockups

For each view identified, create HTML mockup in `pages/`.

**Use dynamic content markers:**

```html
<!-- Data binding -->
<!-- [DATA: item.name] -->

<!-- Loops -->
<!-- [LOOP: items] -->
<tr><td><!-- [DATA: item.name] --></td></tr>
<!-- [END LOOP] -->

<!-- Conditionals -->
<!-- [IF: items.length > 0] -->
  <table>...</table>
<!-- [ELSE] -->
  <div class="empty-state">...</div>
<!-- [END IF] -->

<!-- URLs -->
<a href="<!-- [URL: editItem(item.id)] -->">Edit</a>

<!-- Component includes -->
<!-- [COMPONENT: _status-badge.html, status: item.status] -->

<!-- Form values -->
<input <!-- [VALUE: item.name] --> <!-- [ERROR: errors.name] --> />

<!-- Switch/case -->
<!-- [SWITCH: status] -->
<!-- [CASE: pending] --><span class="badge badge-warning">Pending</span>
<!-- [CASE: active] --><span class="badge badge-info">Active</span>
<!-- [DEFAULT] --><span class="badge badge-ghost">Unknown</span>
<!-- [END SWITCH] -->
```

Include realistic placeholder data to show intended layout.

### Step 5: Extract Reusable Components

Create component files in `components/` with `_` prefix:

- `_status-badge.html` - Status indicator with color mapping
- `_empty-state.html` - Empty collection state with CTA
- `_pagination.html` - Page navigation
- `_delete-modal.html` - Confirmation dialog

## DaisyUI Component Reference

| View Type | Primary Components |
|-----------|-------------------|
| Index/List | table, badge, pagination, dropdown, input (search) |
| Show/Detail | card, stat, badge, breadcrumbs, tabs, timeline |
| Create/Edit | card, fieldset, input, select, textarea, checkbox, button |
| Dashboard | stat, card, table, progress, chart placeholder |
| Report | stats, table, badge, dropdown (filters), export button |
| Modal Dialog | modal, form elements, button group |
| Settings | tabs, toggle, fieldset, alert |

## Theme Guidance

### Light/Professional Theme (default)
- Theme: `nord`, `corporate`, or `lofi`
- Use for: Business applications, dashboards, admin panels

### Dark Theme
- Theme: `dim`, `night`, or `dark`
- Use for: Developer tools, media apps, evening use

## Output Checklist

Before completing mockups, ensure:

- [ ] Each identified view has an HTML mockup in `pages/`
- [ ] Reusable patterns extracted to `components/` with `_` prefix
- [ ] All dynamic content properly marked with `<!-- [MARKER] -->` syntax
- [ ] Mockups use only DaisyUI classes (no custom CSS unless essential)
- [ ] Responsive considerations included (mobile-first classes)
- [ ] Empty states and error states included
- [ ] Accessibility basics covered (labels, ARIA where needed)
