# Create Data Contracts

## Core Responsibility

Document the data requirements for each UI view so implementation knows exactly what variables to provide.

## Workflow

### Step 1: Analyze Each View

For each HTML mockup created, identify:

- **Required variables** (must be provided)
- **Optional variables** (enhance if present)
- **Computed values** (derived from other data)

### Step 2: Generate contracts.md

Write `agent-os/specs/[this-spec]/ui/data-contracts/contracts.md` with:

```markdown
# Data Contracts: [Feature Name]

## Overview

| View | Primary Data | Supporting Data |
|------|--------------|-----------------|
| Index | `items` collection | `pagination`, `filters` |
| Show | `item` object | `canEdit`, `canDelete` |
| Form | `item` (empty/populated) | `statusOptions`, `errors` |

## Common Data Structures

### [Entity] Structure

\`\`\`typescript
interface Entity {
  id: string;
  name: string;
  // ... other fields from spec
}
\`\`\`

## View-Specific Contracts

### pages/index.html

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `items` | `Entity[]` | Yes | Collection to display |
| `pagination` | `Pagination` | Yes | Page state |
| `filters` | `FilterState` | No | Active filters |

### pages/show.html

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `item` | `Entity` | Yes | Item to display |
| `canEdit` | `boolean` | No | Edit permission |
| `canDelete` | `boolean` | No | Delete permission |

### pages/new.html, pages/edit.html

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `item` | `Entity` | Yes | Item data (empty or populated) |
| `errors` | `ValidationErrors` | No | Validation errors |
| `statusOptions` | `SelectOption[]` | Yes | Status dropdown options |
| `isEdit` | `boolean` | Yes | Edit mode flag |

## URL Contracts

| Action | URL Pattern | Method |
|--------|-------------|--------|
| List | `/entities` | GET |
| Show | `/entities/{id}` | GET |
| New | `/entities/new` | GET |
| Create | `/entities` | POST |
| Edit | `/entities/{id}/edit` | GET |
| Update | `/entities/{id}` | PUT/PATCH |
| Delete | `/entities/{id}` | DELETE |

## Flash Messages

| Key | When Used |
|-----|-----------|
| `success` | After successful create/update/delete |
| `error` | After failed operation |
| `warning` | Non-critical alerts |

## Computed Values

| Value | Derivation | Usage |
|-------|------------|-------|
| `isEmpty` | `items.length === 0` | Show empty state |
| `showPagination` | `pagination.totalPages > 1` | Show/hide pagination |
```

### Step 3: Validate Completeness

Ensure every `<!-- [DATA: x] -->` marker in mockups has a corresponding entry in contracts.md.

### Step 4: Add Framework Examples

Include brief implementation examples for common frameworks:

**CFML/Wheels:**
```cfm
<cfset items = model("Entity").findAll(page=params.page, perPage=25)>
```

**Rails/ERB:**
```ruby
@items = Entity.page(params[:page]).per(25)
```

**Laravel/Blade:**
```php
$items = Entity::paginate(25);
```

## Output

The contracts.md file serves as:
1. Implementation reference for controllers/handlers
2. Documentation of expected data structures
3. Validation guide for ensuring views have required data
