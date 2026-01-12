---
name: Wheels Debugging
description: Troubleshoot common Wheels errors and provide debugging guidance. Use when encountering errors, exceptions, or unexpected behavior. Provides error analysis, common solutions, and debugging strategies for Wheels applications.
version: 1.1
---

# Wheels Debugging

## When to Use This Skill

Activate when:
- User encounters an error message or exception
- User says something isn't working as expected
- User mentions debugging, troubleshooting, or fixing
- User sees unexpected behavior in views, forms, or data
- Application crashes or shows error pages
- Tests are failing unexpectedly
- User mentions "error", "exception", "bug", "broken", or "not working"

## Common Model Errors

### "Missing argument name" Error

**Error Message:**
```
Complex object types cannot be converted to simple values.
```

**Cause:** Mixing positional and named arguments in Wheels functions.

**Solution:**
```cfm
// ❌ WRONG - Mixed arguments
hasMany("comments", dependent="delete")

// ✅ CORRECT - All named arguments
hasMany(name="comments", dependent="delete")

// ❌ WRONG - Mixed in routes
.resources("sessions", only="new,create")

// ✅ CORRECT - All named
.resources(name="sessions", only="new,create")
```

### "Can't cast Object type [Query] to [Array]"

**Error Message:**
```
Can't cast Object type [Query] to a value of type [Array]
Detail: Java type of the object is lucee.runtime.type.QueryImpl
```

**Cause:** Using Array functions on query objects returned by Wheels.

**Solution:**
```cfm
// ❌ WRONG - Queries are not arrays
ArrayLen(post.comments())
ArrayAppend(users, newUser)

// ✅ CORRECT - Use query properties/methods
post.comments().recordCount
QueryAddRow(users, newUser)
```

### "key [onCreate,onUpdate] doesn't exist"

**Error Message:**
```
key [onCreate,onUpdate] doesn't exist in the passed in struct
```

**Cause:** Using comma-separated values in validation `when` parameter.

**Solution:**
```cfm
// ❌ WRONG - Comma not supported
validatesConfirmationOf(properties="password", when="onCreate,onUpdate")

// ✅ CORRECT - Omit when (runs on both by default)
validatesConfirmationOf(properties="password")

// ✅ CORRECT - Single value only
validatesConfirmationOf(properties="password", when="onCreate")
```

### "Association not found" Error

**Cause:** Association name typo or not defined in config().

**Debugging Steps:**
1. Check the model's config() function for the association definition
2. Verify association name spelling matches exactly
3. Check the related model exists and has the reciprocal association
4. Ensure the foreign key column exists in the database

```cfm
// In Post.cfc - verify this exists
function config() {
    hasMany(name="comments");  // Check spelling
    belongsTo(name="user");    // Check spelling
}

// In Comment.cfc - verify reciprocal
function config() {
    belongsTo(name="post");    // Must exist
}
```

### "Property [x] does not exist" in Callbacks

**Error Message:**
```
Element X is undefined in THIS
```

**Cause:** Accessing properties before they're set, especially in beforeCreate callbacks.

**Solution:**
```cfm
// ❌ WRONG - Property may not exist yet
function beforeCreate() {
    this.slug = createSlug(this.title);  // title might be undefined
}

// ✅ CORRECT - Check existence first
function beforeCreate() {
    if (structKeyExists(this, "title") && len(this.title)) {
        this.slug = createSlug(this.title);
    }
}
```

### "Invalid salt revision" BCrypt Error

**Error Message:**
```
Invalid salt revision
```

**Cause:** Using BCrypt hash prefix other than `$2a$`.

**Solution:**
```cfm
// ❌ WRONG - $2b$ or $2y$ prefix
passwordHash = "$2b$12$abc..."

// ✅ CORRECT - Use $2a$ prefix for compatibility
passwordHash = "$2a$12$abc..."

// When using jBCrypt:
hashedPassword = BCrypt.hashpw(password, BCrypt.gensalt(12));
// Ensure library generates $2a$ prefix
```

## Common Controller Errors

### "No matching function [RENDERPAGE] found"

**Cause:** Using wrong function name.

**Solution:**
```cfm
// ❌ WRONG - Function doesn't exist
renderPage(action="new")
renderAction("new")

// ✅ CORRECT - Wheels function names
renderView(action="new")
renderView(template="/shared/error")
```

### Filter Not Being Called

**Cause:** Filter inheritance or `except` clause issues.

**Debugging Steps:**
```cfm
// 1. Check filter definition
function config() {
    filters(through="loadPost", only="show,edit,update,delete");
    // Verify action names match exactly
}

// 2. Add debug output to filter
private function loadPost() {
    writeOutput("Filter called for action: #params.action#");
    cfabort;
}

// 3. Check parent controller filters aren't overriding
// Parent filters run BEFORE child filters
```

### Variables Not Available in View

**Cause:** Using `var` keyword in controller makes variable local-only.

**Solution:**
```cfm
// ❌ WRONG - var makes it local to function
function index() {
    var users = model("User").findAll();  // NOT available in view
}

// ✅ CORRECT - Omit var to expose to view
function index() {
    users = model("User").findAll();  // Available in view
}
```

### Redirect Loop

**Cause:** Filter redirects on every request including the redirect target.

**Solution:**
```cfm
// ❌ WRONG - Redirects infinitely
function config() {
    filters(through="requireAuth");  // Applies to ALL actions including login
}

private function requireAuth() {
    if (!loggedIn()) {
        redirectTo(controller="sessions", action="new");  // Also has filter!
    }
}

// ✅ CORRECT - Exclude login actions
function config() {
    filters(through="requireAuth", except="new,create");
}
```

### Flash Messages Not Displaying

**Debugging Steps:**
```cfm
// 1. Verify flash is being set
flashInsert(success="Record saved!");

// 2. Check layout includes flash display
// In layout.cfm:
<cfif flashKeyExists("success")>
    <div class="alert alert-success">#flash("success")#</div>
</cfif>

// 3. Ensure redirect happens (flash survives one request)
// Flash is cleared after being read
redirectTo(action="index");  // Flash available on next request

// 4. Don't use renderView with flash - use redirect
// ❌ WRONG - Flash lost on same request
flashInsert(success="Saved!");
renderView(action="index");  // Flash cleared before display

// ✅ CORRECT
flashInsert(success="Saved!");
redirectTo(action="index");  // Flash available on redirected request
```

## Common View Errors

### CFML Expressions Rendering as Literal Text

**Symptom:** You see `#urlFor(...)#` as text instead of a URL.

**Cause:** CFML expressions outside `<cfoutput>` blocks.

**Solution:**
```cfm
// ❌ WRONG - Not in cfoutput
<a href="#urlFor(action='show')#">View</a>  // Renders as literal text

// ✅ CORRECT - Wrap in cfoutput
<cfoutput>
<a href="#urlFor(action='show')#">View</a>
</cfoutput>

// For layouts - wrap entire HTML
<cfoutput>
<!DOCTYPE html>
<html>
<head><title>#pageTitle#</title></head>
<body>
    #includeContent()#
</body>
</html>
</cfoutput>
```

### "No matching function [EMAILFIELD] found"

**Cause:** Using form helpers that don't exist in Wheels.

**Solution:**
```cfm
// ❌ WRONG - These don't exist in Wheels
#emailField(objectName="user", property="email")#
#urlField(objectName="user", property="website")#
#telField(objectName="user", property="phone")#

// ✅ CORRECT - Use textField with type attribute
#textField(objectName="user", property="email", type="email")#
#textField(objectName="user", property="website", type="url")#
#textField(objectName="user", property="phone", type="tel")#
```

### Duplicate Form Labels

**Symptom:** Labels appear twice (e.g., "Title Title").

**Cause:** Manual label plus Wheels auto-generated label.

**Solution:**
```cfm
// ❌ WRONG - Double labels
<label for="post-title">Title</label>
#textField(objectName="post", property="title")#  // Also generates label!

// ✅ CORRECT - Disable auto-label
<label for="post-title" class="my-label">Title</label>
#textField(objectName="post", property="title", label=false)#

// ✅ OR - Let Wheels handle labels
#textField(objectName="post", property="title", label="Title")#
```

### Query Loop Association Errors

**Error:** Method calls fail inside query loops.

**Solution:**
```cfm
// ❌ WRONG - Can't call method repeatedly in loop
<cfloop query="post.comments()">
    <p>#post.comments().author#</p>
</cfloop>

// ✅ CORRECT - Store in variable first
<cfset comments = post.comments()>
<cfloop query="comments">
    <p>#comments.author#</p>
</cfloop>
```

### Null Property Errors in JavaScript/Alpine.js

**Error:** JavaScript errors when model properties are undefined.

**Solution:**
```cfm
// ❌ WRONG - Undefined on new models
x-data="{ title: '#JSStringFormat(post.title)#' }"

// ✅ CORRECT - Use null coalescing
x-data="{ title: '#JSStringFormat(post.title ?: "")#' }"

// ✅ CORRECT - Or use structKeyExists
x-data="{ title: '#structKeyExists(post, "title") ? JSStringFormat(post.title) : ""#' }"
```

## Common Routing Errors

### "Route Not Found" Error

**Debugging Steps:**
```cfm
// 1. Check route exists in config/routes.cfm
mapper()
    .resources("posts")  // Creates standard CRUD routes
.end();

// 2. Verify URL matches route pattern
// /posts = index
// /posts/new = new
// /posts/123 = show
// /posts/123/edit = edit

// 3. Check HTTP method matches
// GET /posts = index
// POST /posts = create
// DELETE /posts/123 = delete

// 4. Check route order - specific routes before wildcards
mapper()
    .resources("posts")        // Specific first
    .get(name="about", ...)   // Custom routes
    .root(to="home##index")   // Root route
    .wildcard()               // Wildcard LAST
.end();
```

### "Incorrect HTTP Verb" Error

**Error Message:**
```
The posts/1 path does not allow POST requests
```

**Cause:** Using wrong HTTP method for route.

**Solution:**
```cfm
// ❌ WRONG - buttonTo defaults to POST
#buttonTo(controller="posts", action="delete", key=post.id)#

// ✅ CORRECT - Specify method for DELETE
#buttonTo(controller="posts", action="delete", key=post.id, method="delete")#

// ✅ CORRECT - Specify method for PUT/PATCH
#buttonTo(controller="posts", action="update", key=post.id, method="put")#
```

## Common Database Errors

### "Table not found" Error

**Debugging Steps:**
```bash
# 1. Check migration status
wheels dbmigrate info

# 2. Run pending migrations
wheels dbmigrate latest

# 3. Verify table name matches model
# Model: User.cfc → Table: users (pluralized, lowercase)
# Model: OrderItem.cfc → Table: orderitems (lowercase, no underscore by default)
```

### "Column not found" Error

**Debugging Steps:**
```cfm
// 1. Check column exists in database
// Run: DESCRIBE tablename; in database

// 2. Verify property name matches column
// Column: first_name → Property: firstName (camelCase)

// 3. Check for typos in property names
user.firstName  // Not user.firstname or user.first_name

// 4. Run migration to add column if missing
function up() {
    addColumn(table="users", column="firstName", type="string");
}
```

### Migration Errors

**Common Issues:**
```cfm
// 1. Duplicate column
// Error: Column already exists
// Solution: Check if migration already ran, or use conditional
if (!columnExists(table="users", column="newColumn")) {
    addColumn(table="users", column="newColumn", type="string");
}

// 2. Foreign key constraint
// Error: Cannot add foreign key constraint
// Solution: Ensure referenced table/column exists first
function up() {
    // Create parent table first
    createTable(name="categories", ...);

    // Then create child with foreign key
    createTable(name="products", ...);
    addForeignKey(table="products", referenceTable="categories", ...);
}

// 3. Data type mismatch
// Use database-agnostic types
// ❌ WRONG: type="VARCHAR(255)"
// ✅ CORRECT: type="string", limit=255
```

## Debugging Tools and Techniques

### Enable Debug Output

```cfm
// In config/settings.cfm (development only)
set(showDebugInformation=true);
set(showErrorInformation=true);

// Or in config/development/settings.cfm
set(showDebugInformation=true);
```

### Inspect Variables

```cfm
// Quick debug dump
<cfdump var="#post#" label="Post Object">
<cfabort>

// Conditional debugging
<cfif structKeyExists(url, "debug")>
    <cfdump var="#variables#">
</cfif>

// Log to file instead of screen
<cflog file="debug" text="Post ID: #post.id#, Title: #post.title#">
```

### SQL Query Debugging

```cfm
// View SQL in debug output (enable showDebugInformation)
// Red queries = errors
// Yellow queries = slow queries

// Log specific query
posts = model("Post").findAll(where="status = 'active'");
<cflog file="sql" text="#posts.getSQL()#">  // If available

// Use transaction to test without committing
transaction {
    result = model("Post").updateAll(status="archived");
    // Check result
    transaction action="rollback";  // Undo changes
}
```

### Step-by-Step Debugging Workflow

1. **Reproduce the error consistently**
   ```cfm
   // Note exact URL, params, and steps to reproduce
   // URL: /posts/123/edit
   // Error: Variable POSTS is undefined
   ```

2. **Check the controller action**
   ```cfm
   function edit() {
       // Add debug at start
       writeOutput("Edit action called, params.key = #params.key#");
       cfabort;

       post = model("Post").findByKey(key=params.key);
   }
   ```

3. **Check the model query**
   ```cfm
   function edit() {
       post = model("Post").findByKey(key=params.key);

       // Debug model result
       writeOutput("Post found: #isObject(post)#");
       if (isObject(post)) {
           writeDump(post.properties());
       }
       cfabort;
   }
   ```

4. **Check the view**
   ```cfm
   <!--- At top of view --->
   <cfdump var="#isDefined('post')#" label="post defined?">
   <cfif isDefined("post")>
       <cfdump var="#post#" label="post value">
   </cfif>
   <cfabort>
   ```

5. **Check filters**
   ```cfm
   // In filter function
   private function loadPost() {
       writeOutput("Filter loadPost called");
       post = model("Post").findByKey(key=params.key);
       writeOutput("Post in filter: #isObject(post)#");
       cfabort;
   }
   ```

### Browser Testing for UI Debugging

```javascript
// Use Playwright MCP for visual debugging
mcp__playwright__browser_navigate({ url: "http://localhost:3000/posts" })
mcp__playwright__browser_snapshot()  // Get accessibility tree
mcp__playwright__browser_take_screenshot()  // Visual capture

// Check console for JavaScript errors
mcp__playwright__browser_console_messages({ level: "error" })

// Verify form submissions
mcp__playwright__browser_click({ element: "Submit button", ref: "button[type=submit]" })
mcp__playwright__browser_snapshot()  // Check result
```

### Test Failure Debugging

```cfm
// 1. Run single test for focus
wheels test run testSpec=PostModelTest

// 2. Add debug output in test
it("should validate presence of title", function() {
    var post = model("Post").new();

    // Debug
    debug(post);
    debug(post.valid());
    debug(post.allErrors());

    expect(post.valid()).toBeFalse();
});

// 3. Check test database state
// Ensure test database is clean and migrated
wheels dbmigrate latest environment=testing
```

## Error Recovery Patterns

### Graceful Error Handling

```cfm
// Controller error handling
function show() {
    try {
        post = model("Post").findByKey(key=params.key);
        if (!isObject(post)) {
            flashInsert(error="Post not found");
            redirectTo(action="index");
        }
    } catch (any e) {
        // Log error for debugging
        cflog(file="errors", text="Error in posts/show: #e.message#");

        // Show user-friendly message
        flashInsert(error="Something went wrong. Please try again.");
        redirectTo(action="index");
    }
}
```

### Validation Error Display

```cfm
// In view - display all errors
<cfif post.hasErrors()>
    <div class="alert alert-error">
        <ul>
            <cfloop array="#post.allErrors()#" index="error">
                <li>#error.message#</li>
            </cfloop>
        </ul>
    </div>
</cfif>

// Or per-field errors
#textField(objectName="post", property="title")#
<cfif post.hasErrors("title")>
    <span class="error">#post.errorMessageFor("title")#</span>
</cfif>
```

## Related Skills

- `wheels-anti-pattern-detector` - Prevent errors before they happen
- `wheels-refactoring` - Fix code quality issues found during debugging
- `wheels-test-generator` - Write tests to prevent regression
- `wheels-model-generator` - Correct model patterns
- `wheels-controller-generator` - Correct controller patterns

---

**Generated by:** Wheels Debugging Skill v1.1
**Last Updated:** 2025-12-08
