# Data Contracts: [Feature Name]

> This document specifies the data requirements for each UI view.
> Implementation agents should ensure controllers/handlers provide these variables.

## Overview

| View | Primary Data | Supporting Data |
|------|--------------|-----------------|
| Index | `items` collection | `pagination`, `filters`, `statusOptions` |
| Show | `item` object | `relatedItems`, `activityLog` |
| New | `item` (empty/defaults) | `statusOptions`, `errors` |
| Edit | `item` (populated) | `statusOptions`, `errors` |

---

## Common Data Structures

### Item Entity

The primary entity displayed across views.

```typescript
interface Item {
  id: string;                    // Unique identifier
  name: string;                  // Display name
  description?: string;          // Optional description
  status: ItemStatus;            // Current status
  createdAt: DateTime;           // Creation timestamp
  updatedAt: DateTime;           // Last modification
  createdBy?: User;              // Creator reference (optional)
}

type ItemStatus = 'pending' | 'active' | 'completed' | 'cancelled';
```

**Example**:
```json
{
  "id": "item-123",
  "name": "Sample Item",
  "description": "This is a sample item for demonstration.",
  "status": "active",
  "createdAt": "2025-01-12T10:30:00Z",
  "updatedAt": "2025-01-12T14:45:00Z",
  "createdBy": {
    "id": "user-456",
    "name": "John Doe"
  }
}
```

### Pagination

Standard pagination structure for list views.

```typescript
interface Pagination {
  page: number;          // Current page (1-indexed)
  perPage: number;       // Items per page
  totalItems: number;    // Total count
  totalPages: number;    // Calculated total pages
  hasNext: boolean;      // More pages available
  hasPrev: boolean;      // Previous pages available
}
```

**Example**:
```json
{
  "page": 2,
  "perPage": 25,
  "totalItems": 142,
  "totalPages": 6,
  "hasNext": true,
  "hasPrev": true
}
```

### Validation Errors

Structure for form validation errors.

```typescript
interface ValidationErrors {
  [fieldName: string]: string[];  // Field name → array of error messages
}
```

**Example**:
```json
{
  "name": ["Name is required", "Name must be at least 3 characters"],
  "status": ["Please select a valid status"]
}
```

---

## View-Specific Contracts

### Index View (`pages/index.html`)

#### Required Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `items` | `Item[]` | Yes | Collection of items to display |
| `pagination` | `Pagination` | Yes | Pagination state |
| `filters` | `FilterState` | No | Current active filters |
| `statusOptions` | `SelectOption[]` | No | Options for status filter |
| `searchQuery` | `string` | No | Current search term |

#### Filter State

```typescript
interface FilterState {
  status?: ItemStatus;    // Filter by status
  search?: string;        // Search query
  sortBy?: string;        // Sort column
  sortDir?: 'asc' | 'desc';
}
```

#### Select Option

```typescript
interface SelectOption {
  value: string;
  label: string;
  disabled?: boolean;
}
```

**Status Options Example**:
```json
[
  { "value": "", "label": "All Statuses" },
  { "value": "pending", "label": "Pending" },
  { "value": "active", "label": "Active" },
  { "value": "completed", "label": "Completed" },
  { "value": "cancelled", "label": "Cancelled" }
]
```

#### Computed Values

| Value | Derivation | Usage |
|-------|------------|-------|
| `isEmpty` | `items.length === 0` | Show empty state |
| `showPagination` | `pagination.totalPages > 1` | Show/hide pagination |
| `resultRange` | `${start}-${end} of ${total}` | Display "Showing 1-25 of 142" |

---

### Show View (`pages/show.html`)

#### Required Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `item` | `Item` | Yes | The item to display |
| `canEdit` | `boolean` | No | User has edit permission |
| `canDelete` | `boolean` | No | User has delete permission |

#### Optional Variables

| Variable | Type | Description |
|----------|------|-------------|
| `relatedItems` | `Item[]` | Related/linked items |
| `activityLog` | `ActivityEntry[]` | Recent activity history |
| `stats` | `ItemStats` | Computed statistics |

#### Activity Entry

```typescript
interface ActivityEntry {
  id: string;
  action: string;          // 'created', 'updated', 'status_changed'
  description: string;     // Human-readable description
  user: User;              // Who performed the action
  timestamp: DateTime;     // When it happened
  metadata?: object;       // Additional context
}
```

#### Item Stats

```typescript
interface ItemStats {
  [key: string]: {
    value: number | string;
    label: string;
    trend?: 'up' | 'down' | 'neutral';
    changePercent?: number;
  };
}
```

**Example**:
```json
{
  "totalViews": {
    "value": 1234,
    "label": "Total Views",
    "trend": "up",
    "changePercent": 12.5
  },
  "activeTime": {
    "value": "14 days",
    "label": "Time Active",
    "trend": "neutral"
  }
}
```

---

### Form Views (`pages/new.html`, `pages/edit.html`)

#### Required Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `item` | `Item \| Partial<Item>` | Yes | Item data (empty for new, populated for edit) |
| `errors` | `ValidationErrors` | No | Validation errors from failed submission |
| `statusOptions` | `SelectOption[]` | Yes | Options for status select |

#### Form Context

| Variable | Type | Description |
|----------|------|-------------|
| `isEdit` | `boolean` | True if editing existing item |
| `formAction` | `string` | URL to submit form to |
| `formMethod` | `string` | HTTP method (POST or PUT/PATCH) |
| `cancelUrl` | `string` | URL to navigate on cancel |

#### Default Values for New Items

```json
{
  "id": null,
  "name": "",
  "description": "",
  "status": "pending"
}
```

---

## URL Contracts

URLs that views expect to be available:

| Action | URL Pattern | Method | Description |
|--------|-------------|--------|-------------|
| List | `/items` | GET | Index view |
| Show | `/items/{id}` | GET | Detail view |
| New | `/items/new` | GET | Create form |
| Create | `/items` | POST | Create submission |
| Edit | `/items/{id}/edit` | GET | Edit form |
| Update | `/items/{id}` | PUT/PATCH | Update submission |
| Delete | `/items/{id}` | DELETE | Delete item |

### Named Routes (for template helpers)

| Route Name | URL | Parameters |
|------------|-----|------------|
| `items` | `/items` | - |
| `item` | `/items/{id}` | `id` |
| `newItem` | `/items/new` | - |
| `editItem` | `/items/{id}/edit` | `id` |

---

## Flash Messages

Messages passed between requests:

| Key | Type | When Used |
|-----|------|-----------|
| `success` | `string` | After successful create/update/delete |
| `error` | `string` | After failed operation |
| `warning` | `string` | For non-critical alerts |
| `info` | `string` | For informational messages |

**Examples**:
```
success: "Item created successfully"
success: "Item updated successfully"
success: "Item deleted successfully"
error: "Unable to save item. Please try again."
warning: "You have unsaved changes"
```

---

## Implementation Notes

### CFML/Wheels

```cfm
<!--- Controller sets up data --->
<cfset items = model("Item").findAll(
  where = "status = '#params.status#'",
  order = "createdAt DESC",
  page = params.page,
  perPage = 25
)>
<cfset pagination = {
  page = items.currentPage,
  totalPages = items.totalPages,
  totalItems = items.totalRecords
}>

<!--- View accesses --->
<cfloop query="items">
  #items.name#
</cfloop>
```

### Rails/ERB

```ruby
# Controller
@items = Item.where(status: params[:status])
             .order(created_at: :desc)
             .page(params[:page])
             .per(25)

# View accesses via instance variables
<% @items.each do |item| %>
  <%= item.name %>
<% end %>
```

### Laravel/Blade

```php
// Controller
$items = Item::where('status', $request->status)
             ->orderBy('created_at', 'desc')
             ->paginate(25);

return view('items.index', compact('items'));

// View accesses via compact variables
@foreach($items as $item)
  {{ $item->name }}
@endforeach
```

---

## Validation Rules

For reference during implementation:

| Field | Rules |
|-------|-------|
| `name` | Required, min: 3, max: 255 |
| `description` | Optional, max: 2000 |
| `status` | Required, in: pending,active,completed,cancelled |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| [Date] | Initial data contracts created | ui-designer |
