# DaisyUI Standards for Wheels Applications

## Theme Configuration

### Primary Application (nord theme)
- Primary: Professional blue tones
- Use for: Customer-facing interfaces, main application views
- Recommended for: Business applications, dashboards, data-heavy interfaces

### Admin/Platform (dim theme)
- Primary: Dark purple-tinted professional look
- Use for: Admin portals, internal tools, developer interfaces
- Recommended for: Administrative functions, system management

## Component Mapping

| UI Need | DaisyUI Component | Classes |
|---------|-------------------|---------|
| Data tables | table | `table table-zebra` |
| Status indicators | badge | `badge badge-{status}` |
| Action buttons | button | `btn btn-primary btn-sm` |
| Forms | fieldset + input | `fieldset`, `input input-bordered` |
| Cards | card | `card bg-base-100 shadow` |
| Modals | modal | `modal` with `dialog` element |
| Navigation | navbar, menu | `navbar bg-base-100` |
| Alerts/Toasts | alert | `alert alert-{type}` |
| Pagination | join + btn | `join` with `btn btn-sm` |
| Dropdowns | dropdown | `dropdown dropdown-end` |
| Loading states | loading | `loading loading-spinner` |

## Form Patterns

### Standard Input Field
```html
<fieldset class="fieldset">
  <legend class="fieldset-legend">Label</legend>
  <input type="text" class="input input-bordered w-full" />
</fieldset>
```

### Select Field
```html
<fieldset class="fieldset">
  <legend class="fieldset-legend">Label</legend>
  <select class="select select-bordered w-full">
    <option>Option</option>
  </select>
</fieldset>
```

### Textarea Field
```html
<fieldset class="fieldset">
  <legend class="fieldset-legend">Label</legend>
  <textarea class="textarea textarea-bordered w-full" rows="4"></textarea>
</fieldset>
```

### Checkbox Field
```html
<label class="label cursor-pointer justify-start gap-2">
  <input type="checkbox" class="checkbox checkbox-primary" />
  <span class="label-text">Label</span>
</label>
```

### Button Groups
```html
<div class="flex gap-2">
  <button class="btn btn-primary">Save</button>
  <button class="btn btn-ghost">Cancel</button>
</div>
```

### Form Card Layout
```html
<div class="card bg-base-100 shadow max-w-2xl mx-auto">
  <div class="card-body">
    <h2 class="card-title">Form Title</h2>
    <!-- Form fields here -->
    <div class="card-actions justify-end mt-4">
      <button type="submit" class="btn btn-primary">Save</button>
      <a href="[cancel_path]" class="btn btn-ghost">Cancel</a>
    </div>
  </div>
</div>
```

## HTMX Integration Patterns

### Loading States on Buttons
```html
<button class="btn btn-primary" hx-post="/action" hx-indicator="#spinner">
  <span class="loading loading-spinner htmx-indicator" id="spinner"></span>
  Submit
</button>
```

### Loading States in Containers
```html
<div id="content" hx-get="/partial" hx-trigger="load">
  <span class="loading loading-dots loading-lg"></span>
</div>
```

### Swap Transitions
```html
<div hx-get="/items"
     hx-trigger="click"
     hx-swap="innerHTML transition:true"
     class="htmx-swapping:opacity-50">
  <!-- Content -->
</div>
```

### HTMX Indicator CSS
```css
.htmx-indicator { display: none; }
.htmx-request .htmx-indicator { display: inline-flex; }
.htmx-request.htmx-indicator { display: inline-flex; }
```

## Color Usage

| Purpose | Color Class | When to Use |
|---------|-------------|-------------|
| Primary actions | `btn-primary`, `text-primary` | Save, Submit, Create |
| Secondary actions | `btn-secondary` | Less important actions |
| Success states | `badge-success`, `alert-success` | Completed, Active, Paid |
| Warning states | `badge-warning`, `alert-warning` | Pending, Attention needed |
| Error states | `badge-error`, `alert-error` | Failed, Errors, Cancelled |
| Info states | `badge-info`, `alert-info` | Informational messages |
| Neutral | `btn-ghost`, `badge-ghost` | Cancel, Info display |

### Status Badge Mapping
```html
<!-- Generic Status -->
<span class="badge badge-warning">Pending</span>
<span class="badge badge-info">Processing</span>
<span class="badge badge-success">Completed</span>
<span class="badge badge-error">Cancelled</span>

<!-- Boolean Status -->
<span class="badge badge-success">Active</span>
<span class="badge badge-ghost">Inactive</span>

<!-- Payment/Financial Status -->
<span class="badge badge-error">Unpaid</span>
<span class="badge badge-warning">Partial</span>
<span class="badge badge-success">Paid</span>
```

## Responsive Patterns

### Mobile-First Grid
```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- Cards -->
</div>
```

### Responsive Table (with horizontal scroll on mobile)
```html
<div class="overflow-x-auto">
  <table class="table table-zebra">...</table>
</div>
```

### Responsive Stats
```html
<div class="stats stats-vertical lg:stats-horizontal shadow">
  <div class="stat">...</div>
  <div class="stat">...</div>
</div>
```

### Responsive Form Layout
```html
<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
  <fieldset class="fieldset">...</fieldset>
  <fieldset class="fieldset">...</fieldset>
</div>
```

## Table Patterns

### Standard Data Table
```html
<div class="overflow-x-auto">
  <table class="table table-zebra">
    <thead>
      <tr>
        <th>Column 1</th>
        <th>Column 2</th>
        <th class="text-right">Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Value</td>
        <td><span class="badge badge-success">Status</span></td>
        <td class="text-right">
          <div class="flex justify-end gap-1">
            <a href="#" class="btn btn-ghost btn-xs">View</a>
            <a href="#" class="btn btn-ghost btn-xs">Edit</a>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

### Compact Table (for modals/partials)
```html
<table class="table table-xs">
  <thead>
    <tr><th>Item</th><th class="text-right">Amount</th></tr>
  </thead>
  <tbody>...</tbody>
</table>
```

## Modal Patterns

### Standard Modal
```html
<dialog id="my-modal" class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Modal Title</h3>
    <p class="py-4">Modal content goes here.</p>
    <div class="modal-action">
      <form method="dialog">
        <button class="btn">Close</button>
      </form>
    </div>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
```

### Modal with Form
```html
<dialog id="edit-modal" class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Edit Item</h3>
    <form action="/items/update" method="post">
      <!-- Form fields -->
      <div class="modal-action">
        <button type="submit" class="btn btn-primary">Save</button>
        <button type="button" class="btn" onclick="this.closest('dialog').close()">Cancel</button>
      </div>
    </form>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
```

### Opening Modals
```javascript
// Open modal
document.getElementById('my-modal').showModal();

// With HTMX - load content then show
<button hx-get="/modal-content"
        hx-target="#modal-body"
        hx-on::after-swap="document.getElementById('my-modal').showModal()">
  Open
</button>
```

## Alert/Toast Patterns

### Inline Alert
```html
<div class="alert alert-warning">
  <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
  </svg>
  <span>Warning message here</span>
</div>
```

### Toast (positioned)
```html
<div class="toast toast-end">
  <div class="alert alert-success">
    <span>Item saved successfully.</span>
  </div>
</div>
```

## Migration from Custom CSS

### Before (Custom CSS)
```html
<div class="form-card">
  <div class="form-group">
    <label>Name</label>
    <input type="text" class="form-input" />
  </div>
  <div class="form-actions">
    <button class="btn-primary">Save</button>
    <button class="btn-secondary">Cancel</button>
  </div>
</div>
```

### After (DaisyUI)
```html
<div class="card bg-base-100 shadow">
  <div class="card-body">
    <fieldset class="fieldset">
      <legend class="fieldset-legend">Name</legend>
      <input type="text" class="input input-bordered w-full" />
    </fieldset>
    <div class="card-actions justify-end mt-4">
      <button class="btn btn-primary">Save</button>
      <button class="btn btn-ghost">Cancel</button>
    </div>
  </div>
</div>
```

### Common Migration Mappings

| Custom Class | DaisyUI Replacement |
|--------------|---------------------|
| `.form-card` | `.card .card-body` |
| `.form-group` | `.fieldset` |
| `.form-input` | `.input .input-bordered` |
| `.form-select` | `.select .select-bordered` |
| `.btn-primary` | `.btn .btn-primary` |
| `.btn-secondary` | `.btn .btn-ghost` or `.btn .btn-secondary` |
| `.error-messages` | `.alert .alert-error` |
| `.status-badge` | `.badge .badge-{color}` |
| `.data-table` | `.table .table-zebra` |

## Accessibility Considerations

### Focus States
DaisyUI provides built-in focus states. Ensure custom components maintain focus visibility:
```html
<button class="btn btn-primary focus:ring-2 focus:ring-offset-2">
  Accessible Button
</button>
```

### Screen Reader Text
```html
<button class="btn btn-circle btn-ghost">
  <svg>...</svg>
  <span class="sr-only">Close menu</span>
</button>
```

### Color Contrast
DaisyUI themes are designed for accessibility. When customizing:
- Text on `bg-base-100`: Use `text-base-content`
- Text on `bg-primary`: Use `text-primary-content`
- Always test contrast ratios (4.5:1 minimum for normal text)
