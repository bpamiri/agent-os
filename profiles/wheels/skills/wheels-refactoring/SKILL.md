---
name: Wheels Refactoring
description: Refactor Wheels code for better performance, security, and maintainability. Use when optimizing code, fixing anti-patterns, improving performance, or enhancing security. Provides refactoring patterns and best practices.
version: 1.1
---

# Wheels Refactoring

## When to Use This Skill

Activate when:
- User wants to optimize slow code or queries
- User mentions N+1 queries or performance issues
- User wants to improve code quality or maintainability
- User is fixing security vulnerabilities
- User mentions refactoring, optimization, or cleanup
- User has duplicate code they want to consolidate
- User wants to move logic to appropriate layers (fat models, skinny controllers)

## Performance Refactoring

### N+1 Query Problem

The most common performance issue in Wheels applications.

**Before (N+1 queries):**
```cfm
<!--- Controller --->
posts = model("Post").findAll();

<!--- View: executes 1 + N queries --->
<cfloop query="posts">
    <p>Author: #posts.user().name#</p>  <!--- Query for EACH post! --->
    <p>Comments: #posts.comments().recordCount#</p>  <!--- Another query per post! --->
</cfloop>
```

**After (2 queries total):**
```cfm
<!--- Controller: eager load associations --->
posts = model("Post").findAll(include="user,comments");

<!--- View: no additional queries --->
<cfloop query="posts">
    <p>Author: #posts.userName#</p>  <!--- Already loaded --->
    <p>Comments: #posts.commentsCount#</p>  <!--- Use counter cache or subquery --->
</cfloop>
```

### Deep Eager Loading

**Before:**
```cfm
orders = model("Order").findAll(include="customer");
// lineItems and product not loaded - will cause N+1
```

**After:**
```cfm
orders = model("Order").findAll(include="customer,lineItems(product)");
// Nested eager loading - all data in minimal queries
```

### Selective Column Loading

**Before:**
```cfm
// Loads ALL columns including large text fields
users = model("User").findAll();
```

**After:**
```cfm
// Only load columns needed for display
users = model("User").findAll(select="id,name,email,createdAt");
```

### Pagination for Large Datasets

**Before:**
```cfm
// Loads entire table into memory
allProducts = model("Product").findAll();
```

**After:**
```cfm
// Paginated loading
products = model("Product").findAll(
    page=params.page ?: 1,
    perPage=25,
    order="name ASC"
);
```

### Counter Cache Pattern

**Before:**
```cfm
// Query to count comments for every post display
<cfloop query="posts">
    <span>#posts.comments().recordCount# comments</span>
</cfloop>
```

**After:**
```cfm
// Add commentsCount column to posts table via migration
// Then update count in Comment model callbacks

// In Comment.cfc
function afterCreate() {
    updatePostCommentCount();
}

function afterDelete() {
    updatePostCommentCount();
}

private function updatePostCommentCount() {
    post = this.post();
    post.update(commentsCount=post.comments().recordCount);
}

// In view - no query needed
<cfloop query="posts">
    <span>#posts.commentsCount# comments</span>
</cfloop>
```

### Query Caching

**Before:**
```cfm
// Same query executed multiple times per request
function getCategories() {
    return model("Category").findAll(order="name");
}
```

**After:**
```cfm
// Cache in request scope
function getCategories() {
    if (!structKeyExists(request, "cachedCategories")) {
        request.cachedCategories = model("Category").findAll(order="name");
    }
    return request.cachedCategories;
}

// Or use Wheels caching
categories = model("Category").findAll(order="name", cache=60); // 60 seconds
```

### Batch Processing

**Before:**
```cfm
// Individual saves cause many database round trips
for (item in items) {
    model("OrderItem").create(item);
}
```

**After:**
```cfm
// Use transactions for batch operations
transaction {
    for (item in items) {
        model("OrderItem").create(item);
    }
}

// Or use bulk insert if available
model("OrderItem").createAll(items);
```

## Security Refactoring

### Parameter Verification

**Before:**
```cfm
function show() {
    // No validation - accepts any params.key value
    post = model("Post").findByKey(key=params.key);
}
```

**After:**
```cfm
function config() {
    // Verify params before action executes
    verifies(only="show,edit,update,delete", params="key", paramsTypes="integer");
}

function show() {
    post = model("Post").findByKey(key=params.key);
    if (!isObject(post)) {
        flashInsert(error="Post not found");
        redirectTo(action="index");
    }
}
```

### Mass Assignment Protection

**Before:**
```cfm
// Vulnerable to mass assignment - user could set isAdmin=true
user = model("User").new(params.user);
user.save();
```

**After:**
```cfm
// Whitelist allowed properties
allowedParams = {
    name = params.user.name,
    email = params.user.email,
    password = params.user.password
};
user = model("User").new(allowedParams);
user.save();

// Or use model-level protection
// In User.cfc
function config() {
    property(name="isAdmin", setter=false); // Can't be mass assigned
}
```

### Tenant Isolation (Multi-Tenant Apps)

**Before:**
```cfm
// Forgot tenant scoping - data leak!
customers = model("Customer").findAll();
```

**After:**
```cfm
// Always scope by tenant
customers = model("Customer").findAll(where="tenantId = #session.currentTenant.id#");

// Better: Use TenantModel base class
// In Customer.cfc
component extends="TenantModel" {
    // TenantModel automatically adds tenantId to all queries
}
```

### Authorization Checks

**Before:**
```cfm
function edit() {
    post = model("Post").findByKey(key=params.key);
    // Anyone can edit any post!
}
```

**After:**
```cfm
function edit() {
    post = model("Post").findByKey(key=params.key);

    // Check ownership or admin status
    if (post.userId != session.currentUser.id && !isAdmin()) {
        flashInsert(error="You don't have permission to edit this post");
        redirectTo(action="show", key=params.key);
    }
}
```

### SQL Injection Prevention

**Before:**
```cfm
// String concatenation - VULNERABLE!
users = model("User").findAll(
    where="status = '#params.status#'"
);
```

**After:**
```cfm
// Wheels automatically parameterizes, but validate input
if (listFindNoCase("active,inactive,pending", params.status)) {
    users = model("User").findAll(
        where="status = '#params.status#'"
    );
}

// Or use struct-based where for complex queries
users = model("User").findAll(
    where={status = params.status}
);
```

## Code Quality Refactoring

### Fat Models, Skinny Controllers

**Before (Fat Controller):**
```cfm
function create() {
    user = model("User").new(params.user);
    user.password = hash(user.password, "SHA-512");
    user.confirmationToken = createUUID();
    user.status = "pending";

    if (user.save()) {
        // Send welcome email
        sendMail(
            to = user.email,
            subject = "Welcome to our site",
            template = "/emails/welcome"
        );

        // Create audit log
        model("AuditLog").create(
            userId = user.id,
            action = "registration",
            details = "User registered"
        );

        flashInsert(success="Account created!");
        redirectTo(action="show", key=user.key());
    } else {
        renderView(action="new");
    }
}
```

**After (Skinny Controller, Fat Model):**
```cfm
// Controller - only handles HTTP concerns
function create() {
    user = model("User").new(params.user);
    if (user.save()) {
        flashInsert(success="Account created!");
        redirectTo(action="show", key=user.key());
    } else {
        renderView(action="new");
    }
}

// User.cfc - handles business logic
function config() {
    beforeCreate("prepareForRegistration");
    afterCreate("sendWelcomeEmail,createAuditLog");
}

private function prepareForRegistration() {
    this.password = hashPassword(this.password);
    this.confirmationToken = createUUID();
    this.status = "pending";
}

private function sendWelcomeEmail() {
    sendMail(
        to = this.email,
        subject = "Welcome to our site",
        template = "/emails/welcome"
    );
}

private function createAuditLog() {
    model("AuditLog").create(
        userId = this.id,
        action = "registration",
        details = "User registered"
    );
}
```

### Extract Reusable Methods

**Before:**
```cfm
// Same price calculation in multiple places
function show() {
    order = model("Order").findByKey(key=params.key, include="lineItems");

    subtotal = 0;
    for (item in order.lineItems()) {
        subtotal += item.quantity * item.unitPrice;
    }
    taxRate = 0.08;
    tax = subtotal * taxRate;
    total = subtotal + tax;
}
```

**After:**
```cfm
// In Order.cfc
function calculateSubtotal() {
    var subtotal = 0;
    var items = this.lineItems();
    for (var i = 1; i <= items.recordCount; i++) {
        subtotal += items.quantity[i] * items.unitPrice[i];
    }
    return subtotal;
}

function calculateTax() {
    return calculateSubtotal() * getTaxRate();
}

function calculateTotal() {
    return calculateSubtotal() + calculateTax();
}

private function getTaxRate() {
    return 0.08; // Could look up by location
}

// Controller - clean and simple
function show() {
    order = model("Order").findByKey(key=params.key, include="lineItems");
}

// View
<p>Subtotal: #dollarFormat(order.calculateSubtotal())#</p>
<p>Tax: #dollarFormat(order.calculateTax())#</p>
<p>Total: #dollarFormat(order.calculateTotal())#</p>
```

### DRY View Refactoring with Partials

**Before:**
```cfm
<!--- index.cfm --->
<cfloop query="posts">
    <div class="card">
        <h3>#posts.title#</h3>
        <p>By #posts.authorName# on #dateFormat(posts.createdAt, "mmm d, yyyy")#</p>
        <p>#left(posts.body, 200)#...</p>
        <a href="#urlFor(action='show', key=posts.id)#">Read more</a>
    </div>
</cfloop>

<!--- Same card structure in sidebar, related posts, etc. --->
```

**After:**
```cfm
<!--- index.cfm --->
<cfloop query="posts">
    #includePartial(partial="postCard", post=posts)#
</cfloop>

<!--- _postCard.cfm --->
<div class="card">
    <h3>#arguments.post.title#</h3>
    <p>By #arguments.post.authorName# on #dateFormat(arguments.post.createdAt, "mmm d, yyyy")#</p>
    <p>#left(arguments.post.body, 200)#...</p>
    <a href="#urlFor(action='show', key=arguments.post.id)#">Read more</a>
</div>
```

### Model Scopes

**Before:**
```cfm
// Same where clause repeated everywhere
activeUsers = model("User").findAll(where="status = 'active' AND deletedAt IS NULL");
activeAdmins = model("User").findAll(where="status = 'active' AND deletedAt IS NULL AND role = 'admin'");
```

**After:**
```cfm
// In User.cfc - define reusable scopes
function active() {
    return model("User").findAll(where="status = 'active' AND deletedAt IS NULL");
}

function activeAdmins() {
    return model("User").findAll(where="status = 'active' AND deletedAt IS NULL AND role = 'admin'");
}

// Usage
activeUsers = model("User").active();
activeAdmins = model("User").activeAdmins();
```

### Filter Consolidation

**Before:**
```cfm
// Same authentication check in every controller
component extends="Controller" {
    function config() {
        filters(through="requireAuth");
    }

    private function requireAuth() {
        if (!structKeyExists(session, "currentUser")) {
            flashInsert(error="Please log in");
            redirectTo(controller="sessions", action="new");
        }
    }
}
```

**After:**
```cfm
// BaseController.cfc
component extends="Controller" {
    function config() {
        filters(through="requireAuth", except="");
    }

    private function requireAuth() {
        if (!structKeyExists(session, "currentUser")) {
            flashInsert(error="Please log in");
            redirectTo(controller="sessions", action="new");
        }
    }

    private function isAdmin() {
        return session.currentUser.role == "admin";
    }
}

// Other controllers just extend BaseController
component extends="BaseController" {
    function config() {
        super.config();
        // Add controller-specific filters
    }
}
```

## Database Refactoring

### Add Missing Indexes

**Before (slow queries):**
```sql
-- Query plan shows full table scan
SELECT * FROM orders WHERE customerId = 123 AND status = 'pending';
```

**After (indexed):**
```cfm
// Migration to add index
function up() {
    addIndex(table="orders", columns="customerId,status");
}

function down() {
    removeIndex(table="orders", indexName="idx_orders_customerId_status");
}
```

### Normalize Repeated Data

**Before:**
```cfm
// Address duplicated in multiple tables
orders table: street, city, state, zip
customers table: street, city, state, zip
suppliers table: street, city, state, zip
```

**After:**
```cfm
// Separate addresses table
addresses table: id, street, city, state, zip, addressableType, addressableId

// Models use polymorphic association
// In Order.cfc
belongsTo(name="shippingAddress", modelName="Address", foreignKey="shippingAddressId");
belongsTo(name="billingAddress", modelName="Address", foreignKey="billingAddressId");
```

## Related Skills

- `wheels-anti-pattern-detector` - Detect issues before refactoring
- `wheels-debugging` - Debug issues found during refactoring
- `wheels-test-generator` - Write tests before refactoring (TDD)
- `wheels-model-generator` - Generate new models after extraction

---

**Generated by:** Wheels Refactoring Skill v1.1
**Last Updated:** 2025-12-08
