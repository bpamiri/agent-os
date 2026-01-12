# View Template Standards

## File Organization

```
app/views/
├── [controller]/
│   ├── index.cfm      # List view
│   ├── show.cfm       # Detail view
│   ├── new.cfm        # Create form
│   ├── edit.cfm       # Edit form
│   ├── _form.cfm      # Shared form partial
│   ├── _table.cfm     # Table partial for index
│   ├── _row.cfm       # Single row (HTMX updates)
│   └── _modal.cfm     # Modal content
└── layout.cfm         # Main layout
```

## Naming Conventions

- **Partials**: Prefix with underscore (`_form.cfm`)
- **HTMX targets**: Use descriptive IDs (`#line-items-table`)
- **Modal partials**: `_[action]_modal.cfm` (e.g., `_waive_core_modal.cfm`)
- **Section partials**: `_[section]_section.cfm` (e.g., `_totals_section.cfm`)

## View Type Patterns

### Index View (List)

Standard list view with header, filters, table, and pagination:

```cfm
<div class="flex justify-between items-center mb-4">
  <h1 class="text-2xl font-bold">[Resources]</h1>
  <a href="#urlFor(route='new[Resource]')#" class="btn btn-primary">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
      <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
    </svg>
    Add New
  </a>
</div>

<!--- Filters --->
<div class="flex flex-wrap gap-2 mb-4">
  <select class="select select-bordered select-sm" name="status"
          hx-get="#urlFor(route='[resources]')#"
          hx-target="#[resource]-table"
          hx-include="[name='search']">
    <option value="">All Statuses</option>
    <option value="active">Active</option>
    <option value="inactive">Inactive</option>
  </select>
  <input type="search" name="search"
         class="input input-bordered input-sm"
         placeholder="Search..."
         hx-get="#urlFor(route='[resources]')#"
         hx-target="#[resource]-table"
         hx-trigger="keyup changed delay:300ms"
         hx-include="[name='status']" />
</div>

<!--- Table --->
<div id="[resource]-table">
  <cfinclude template="_table.cfm">
</div>
```

### Table Partial (_table.cfm)

```cfm
<div class="overflow-x-auto">
  <table class="table table-zebra">
    <thead>
      <tr>
        <th>Name</th>
        <th>Status</th>
        <th>Created</th>
        <th class="text-right">Actions</th>
      </tr>
    </thead>
    <tbody>
      <cfif items.recordCount>
        <cfloop query="items">
          <cfinclude template="_row.cfm">
        </cfloop>
      <cfelse>
        <tr>
          <td colspan="4" class="text-center py-8 text-base-content/60">
            No [resources] found.
          </td>
        </tr>
      </cfif>
    </tbody>
  </table>
</div>

<!--- Pagination --->
<cfif totalPages GT 1>
  <div class="flex justify-center mt-4">
    <div class="join">
      <cfloop from="1" to="#totalPages#" index="p">
        <a href="#urlFor(route='[resources]', params='page=#p#')#"
           class="join-item btn btn-sm <cfif p EQ currentPage>btn-active</cfif>">
          #p#
        </a>
      </cfloop>
    </div>
  </div>
</cfif>
```

### Row Partial (_row.cfm)

```cfm
<tr id="[resource]-row-#items.id#">
  <td>
    <a href="#urlFor(route='[resource]', key=items.id)#" class="link link-hover">
      #items.name#
    </a>
  </td>
  <td>
    <span class="badge badge-#items.status EQ 'active' ? 'success' : 'ghost'#">
      #items.status#
    </span>
  </td>
  <td>#dateFormat(items.createdAt, 'mmm d, yyyy')#</td>
  <td class="text-right">
    <div class="flex justify-end gap-1">
      <a href="#urlFor(route='[resource]', key=items.id)#"
         class="btn btn-ghost btn-xs">View</a>
      <a href="#urlFor(route='edit[Resource]', key=items.id)#"
         class="btn btn-ghost btn-xs">Edit</a>
      <button class="btn btn-ghost btn-xs text-error"
              hx-delete="#urlFor(route='[resource]', key=items.id)#"
              hx-confirm="Delete this [resource]?"
              hx-target="#[resource]-row-#items.id#"
              hx-swap="outerHTML">
        Delete
      </button>
    </div>
  </td>
</tr>
```

### Show View (Detail)

```cfm
<!--- Breadcrumb --->
<div class="text-sm breadcrumbs mb-4">
  <ul>
    <li><a href="#urlFor(route='[resources]')#">[Resources]</a></li>
    <li>#[resource].name#</li>
  </ul>
</div>

<div class="card bg-base-100 shadow">
  <div class="card-body">
    <!--- Header --->
    <div class="flex justify-between items-start">
      <div>
        <h2 class="card-title text-2xl">#[resource].name#</h2>
        <p class="text-base-content/60">#[resource].description#</p>
      </div>
      <span class="badge badge-lg badge-#statusColor#">#[resource].status#</span>
    </div>

    <!--- Stats/Summary --->
    <div class="stats shadow mt-4">
      <div class="stat">
        <div class="stat-title">Total</div>
        <div class="stat-value">#dollarFormat([resource].total)#</div>
      </div>
      <div class="stat">
        <div class="stat-title">Items</div>
        <div class="stat-value">#[resource].itemCount#</div>
      </div>
      <div class="stat">
        <div class="stat-title">Created</div>
        <div class="stat-value text-lg">#dateFormat([resource].createdAt, 'mmm d, yyyy')#</div>
      </div>
    </div>

    <!--- Details Section --->
    <div class="divider">Details</div>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div>
        <span class="text-base-content/60">Field Label</span>
        <p class="font-medium">#[resource].fieldValue#</p>
      </div>
      <!--- More fields... --->
    </div>

    <!--- Actions --->
    <div class="card-actions justify-end mt-4">
      <a href="#urlFor(route='edit[Resource]', key=[resource].id)#"
         class="btn btn-primary">Edit</a>
      <button class="btn btn-error btn-outline"
              hx-delete="#urlFor(route='[resource]', key=[resource].id)#"
              hx-confirm="Delete this [resource]?">
        Delete
      </button>
    </div>
  </div>
</div>
```

### Form View (New/Edit)

**new.cfm:**
```cfm
<div class="text-sm breadcrumbs mb-4">
  <ul>
    <li><a href="#urlFor(route='[resources]')#">[Resources]</a></li>
    <li>New</li>
  </ul>
</div>

<div class="card bg-base-100 shadow max-w-2xl mx-auto">
  <div class="card-body">
    <h2 class="card-title">Create [Resource]</h2>

    #startFormTag(route="[resources]", method="post")#
      <cfinclude template="_form.cfm">

      <div class="card-actions justify-end mt-6">
        <a href="#urlFor(route='[resources]')#" class="btn btn-ghost">Cancel</a>
        <button type="submit" class="btn btn-primary">Create [Resource]</button>
      </div>
    #endFormTag()#
  </div>
</div>
```

**edit.cfm:**
```cfm
<div class="text-sm breadcrumbs mb-4">
  <ul>
    <li><a href="#urlFor(route='[resources]')#">[Resources]</a></li>
    <li><a href="#urlFor(route='[resource]', key=[resource].id)#">#[resource].name#</a></li>
    <li>Edit</li>
  </ul>
</div>

<div class="card bg-base-100 shadow max-w-2xl mx-auto">
  <div class="card-body">
    <h2 class="card-title">Edit [Resource]</h2>

    #startFormTag(route="[resource]", key=[resource].id, method="patch")#
      <cfinclude template="_form.cfm">

      <div class="card-actions justify-end mt-6">
        <a href="#urlFor(route='[resource]', key=[resource].id)#" class="btn btn-ghost">Cancel</a>
        <button type="submit" class="btn btn-primary">Save Changes</button>
      </div>
    #endFormTag()#
  </div>
</div>
```

### Form Partial (_form.cfm)

```cfm
<!--- Display validation errors --->
<cfif structKeyExists([resource], "errors") AND NOT structIsEmpty([resource].errors)>
  <div class="alert alert-error mb-4">
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <div>
      <h3 class="font-bold">Please correct the following errors:</h3>
      <ul class="list-disc list-inside">
        <cfloop collection="#[resource].errors#" item="field">
          <cfloop array="#[resource].errors[field]#" index="msg">
            <li>#msg#</li>
          </cfloop>
        </cfloop>
      </ul>
    </div>
  </div>
</cfif>

<!--- Form fields --->
<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
  <fieldset class="fieldset">
    <legend class="fieldset-legend">Name <span class="text-error">*</span></legend>
    #textField(objectName="[resource]", property="name", class="input input-bordered w-full")#
  </fieldset>

  <fieldset class="fieldset">
    <legend class="fieldset-legend">Status</legend>
    #selectField(objectName="[resource]", property="status",
                 options=statusOptions, class="select select-bordered w-full")#
  </fieldset>
</div>

<fieldset class="fieldset mt-4">
  <legend class="fieldset-legend">Description</legend>
  #textArea(objectName="[resource]", property="description",
            class="textarea textarea-bordered w-full", rows="4")#
</fieldset>

<label class="label cursor-pointer justify-start gap-2 mt-4">
  #checkBox(objectName="[resource]", property="isActive", class="checkbox checkbox-primary")#
  <span class="label-text">Active</span>
</label>
```

### Modal Pattern

**Modal Trigger Button:**
```cfm
<button class="btn btn-primary btn-sm"
        hx-get="#urlFor(route='[modalContent]', key=id)#"
        hx-target="#modal-content"
        hx-on::after-swap="document.getElementById('action-modal').showModal()">
  Open Modal
</button>
```

**Modal Container (in layout or page):**
```cfm
<dialog id="action-modal" class="modal">
  <div class="modal-box" id="modal-content">
    <!--- Content loaded via HTMX --->
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
```

**Modal Content Partial (_action_modal.cfm):**
```cfm
<h3 class="font-bold text-lg">Modal Title</h3>

#startFormTag(route="[action]", method="post")#
  <div class="py-4">
    <fieldset class="fieldset">
      <legend class="fieldset-legend">Field</legend>
      #textField(objectName="item", property="value", class="input input-bordered w-full")#
    </fieldset>
  </div>

  <div class="modal-action">
    <button type="submit" class="btn btn-primary">Confirm</button>
    <button type="button" class="btn" onclick="this.closest('dialog').close()">Cancel</button>
  </div>
#endFormTag()#
```

## HTMX Patterns

### Inline Edit
```cfm
<td hx-get="#urlFor(route='editCell', key=id)#"
    hx-trigger="dblclick"
    hx-swap="innerHTML">
  #value#
</td>
```

### Delete with Row Removal
```cfm
<button class="btn btn-error btn-sm"
        hx-delete="#urlFor(route='[resource]', key=id)#"
        hx-confirm="Delete this [resource]?"
        hx-target="closest tr"
        hx-swap="outerHTML swap:500ms">
  Delete
</button>
```

### Partial Refresh
```cfm
<button class="btn btn-sm"
        hx-get="#urlFor(route='refreshSection')#"
        hx-target="#section-container"
        hx-swap="innerHTML">
  Refresh
</button>
```

### Form Submission with Response
```cfm
#startFormTag(route="[resource]", method="post",
              hx-post=urlFor(route="[resources]"),
              hx-target="#form-result",
              hx-swap="innerHTML")#
  <!--- Form fields --->
  <button type="submit" class="btn btn-primary">
    <span class="loading loading-spinner htmx-indicator"></span>
    Save
  </button>
#endFormTag()#
<div id="form-result"></div>
```

## Layout Integration

### Page Titles
Set page title in controller or at top of view:
```cfm
<cfset pageTitle = "[Resource] List">
```

### Flash Messages (in layout)
```cfm
<cfif structKeyExists(flash, "success")>
  <div class="alert alert-success mb-4">
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
    </svg>
    <span>#flash.success#</span>
  </div>
</cfif>
```

## Wheels Form Helper Integration

### Using Wheels Helpers with DaisyUI Classes
```cfm
<!--- Text Input --->
#textField(objectName="user", property="email",
           class="input input-bordered w-full",
           placeholder="Enter email")#

<!--- Select --->
#selectField(objectName="user", property="role",
             options=roleOptions,
             class="select select-bordered w-full",
             includeBlank="Select role...")#

<!--- Textarea --->
#textArea(objectName="user", property="bio",
          class="textarea textarea-bordered w-full",
          rows="4")#

<!--- Checkbox --->
#checkBox(objectName="user", property="isActive",
          class="checkbox checkbox-primary")#

<!--- Radio Buttons --->
<cfloop array="#options#" index="opt">
  <label class="label cursor-pointer justify-start gap-2">
    #radioButton(objectName="item", property="type",
                 tagValue=opt.value, class="radio radio-primary")#
    <span class="label-text">#opt.label#</span>
  </label>
</cfloop>
```

## Report Views

### Report Header with Filters
```cfm
<div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-6">
  <div>
    <h1 class="text-2xl font-bold">[Report Name]</h1>
    <p class="text-base-content/60">Description of report</p>
  </div>

  <div class="flex flex-wrap gap-2">
    <select class="select select-bordered select-sm" name="period">
      <option value="today">Today</option>
      <option value="week">This Week</option>
      <option value="month">This Month</option>
    </select>
    <a href="#urlFor(route='exportReport', params='format=csv')#"
       class="btn btn-outline btn-sm">
      Export CSV
    </a>
  </div>
</div>
```

### Summary Stats for Reports
```cfm
<div class="stats stats-vertical lg:stats-horizontal shadow mb-6 w-full">
  <div class="stat">
    <div class="stat-title">Total Sales</div>
    <div class="stat-value text-primary">#dollarFormat(totalSales)#</div>
    <div class="stat-desc">#numberFormat(orderCount)# orders</div>
  </div>
  <div class="stat">
    <div class="stat-title">Average Order</div>
    <div class="stat-value">#dollarFormat(avgOrder)#</div>
  </div>
  <div class="stat">
    <div class="stat-title">Top Category</div>
    <div class="stat-value text-lg">#topCategory#</div>
    <div class="stat-desc">#numberFormat(topCategoryPercent, '0.0')#% of sales</div>
  </div>
</div>
```
