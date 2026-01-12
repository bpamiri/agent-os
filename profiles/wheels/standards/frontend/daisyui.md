## DaisyUI Standards for Server-Rendered Templates

### Theme Selection

Use appropriate DaisyUI themes based on application context:

| Theme | Use For |
|-------|---------|
| `nord`, `corporate` | Professional business applications |
| `dim`, `night` | Developer tools, admin panels |
| `lofi` | Minimal, content-focused interfaces |
| `cupcake` | Friendly, consumer-facing applications |

### Component Usage Patterns

**Tables**: Use `table table-zebra` for data lists. Always include proper `<thead>` with `<th>` elements for accessibility.

**Forms**: Use `fieldset` with `legend` for grouped inputs. Apply `form-control` wrapper for labels and inputs.

**Buttons**: Use semantic variants:
- `btn-primary` - Main actions (Save, Create)
- `btn-ghost` - Secondary actions (Cancel, Back)
- `btn-error` - Destructive actions (Delete)
- `btn-sm` - Table row actions

**Status Indicators**: Use `badge` with semantic colors:
- `badge-warning` - Pending states
- `badge-info` - Active/In Progress
- `badge-success` - Completed/Success
- `badge-error` - Failed/Cancelled

**Cards**: Use `card` with `card-body` for content sections. Add `shadow-lg` for elevated appearance.

**Modals**: Use `modal` with `modal-box` for dialogs. Always include close button and escape key handling.

### Server Template Integration

When implementing DaisyUI mockups in server templates:

1. **Preserve Class Structure**: Copy DaisyUI classes exactly from mockups
2. **Replace Markers**: Convert `<!-- [DATA: x] -->` to your template syntax
3. **Extract Partials**: Move `_component.html` files to framework's partial location
4. **Maintain Accessibility**: Keep all ARIA attributes from mockups

### Marker Syntax Translation

| Marker | CFML | ERB | Blade |
|--------|------|-----|-------|
| `<!-- [DATA: x] -->` | `#x#` | `<%= x %>` | `{{ $x }}` |
| `<!-- [LOOP: items] -->` | `<cfloop>` | `<% @items.each do \|item\| %>` | `@foreach($items as $item)` |
| `<!-- [IF: cond] -->` | `<cfif cond>` | `<% if cond %>` | `@if($cond)` |
| `<!-- [URL: route] -->` | `#urlFor()#` | `<%= path %>` | `{{ route() }}` |

### Responsive Design

Apply mobile-first responsive classes:
- Default styles for mobile (< 768px)
- `md:` prefix for tablet (>= 768px)
- `lg:` prefix for desktop (>= 1024px)

Example: `class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3"`

### Empty States

Always include empty state handling when collections may be empty:

```html
<!-- [IF: items.length > 0] -->
  <table>...</table>
<!-- [ELSE] -->
  <!-- [COMPONENT: _empty-state.html] -->
<!-- [END IF] -->
```

### Form Validation Display

Show validation errors using DaisyUI patterns:

```html
<input class="input input-bordered <!-- [IF: errors.name] -->input-error<!-- [END IF] -->" />
<!-- [IF: errors.name] -->
<div class="label">
  <span class="label-text-alt text-error"><!-- [DATA: errors.name] --></span>
</div>
<!-- [END IF] -->
```

### Action Dropdowns

Use dropdown for row actions in tables:

```html
<div class="dropdown dropdown-end">
  <div tabindex="0" role="button" class="btn btn-ghost btn-sm">...</div>
  <ul class="dropdown-content menu bg-base-100 rounded-box z-10 w-40 p-2 shadow-lg">
    <li><a href="...">View</a></li>
    <li><a href="...">Edit</a></li>
    <li><a class="text-error" href="...">Delete</a></li>
  </ul>
</div>
```
