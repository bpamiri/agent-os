# Browser Feature Test

Comprehensive end-to-end browser testing for recently implemented features. Tests all CRUD operations, menu routes, and UX interactions. Use after completing a feature implementation to verify everything works correctly in the browser.

## When to Use

Invoke this command when:
- You've just completed implementing a feature
- You want to verify all CRUD operations work
- You need to test menu routes and navigation
- You want to create sample records and test the UX
- At the end of an implementation session to validate work

## Usage

```
/agent-os:browser-feature-test [feature-name]
```

If no feature name is provided, the command will detect the most recently modified spec.

## Testing Process

### Phase 1: Context Gathering

1. **Identify Feature to Test**
   - If feature name provided, locate spec in `agent-os/specs/`
   - Otherwise, find most recently modified spec folder
   - Read `spec.md` and `tasks.md` to understand what was built

2. **Identify Routes and Controllers**
   - Read `config/routes.cfm` to find all routes for the feature
   - Identify CRUD endpoints (index, show, new, create, edit, update, delete)
   - Note any custom routes or actions

3. **Get Server Port**
   - Check if Wheels server is running
   - Get the port number for navigation

### Phase 2: Browser Testing Setup

1. **Get Browser Tab Context**
   ```javascript
   mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty=true)
   ```

2. **Create New Tab for Testing**
   ```javascript
   mcp__claude-in-chrome__tabs_create_mcp()
   ```

3. **Start GIF Recording** (for documentation)
   ```javascript
   mcp__claude-in-chrome__gif_creator(action="start_recording", tabId=TAB_ID)
   mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
   ```

### Phase 3: Authentication (if required)

1. **Navigate to Login**
   ```javascript
   mcp__claude-in-chrome__navigate(url="http://localhost:PORT/login", tabId=TAB_ID)
   ```

2. **Enter Test Credentials**
   - Use demo account: `demo@bookkeeper.local` / `password123`
   - Or create a new test user via registration

3. **Verify Login Success**
   - Screenshot after login
   - Verify redirect to expected page

### Phase 4: Feature Testing

#### 4.1 Navigation/Menu Testing
- Click through all menu items related to the feature
- Verify each route loads correctly
- Screenshot each page
- Check for any error messages

#### 4.2 Index/List View Testing
- Navigate to the index page
- Verify empty state message if no records
- Check pagination if applicable
- Screenshot the list view

#### 4.3 Create Operation Testing
- Navigate to "New" form
- Fill in all required fields with sample data
- Submit the form
- Verify success message
- Verify redirect to show/index page
- Screenshot at each step

#### 4.4 Read/Show Operation Testing
- Navigate to the show page for created record
- Verify all fields display correctly
- Check associated data displays
- Screenshot the detail view

#### 4.5 Update Operation Testing
- Navigate to edit form
- Modify some fields
- Submit the form
- Verify success message
- Verify changes are persisted
- Screenshot before and after

#### 4.6 Delete Operation Testing (if applicable)
- Navigate to record with delete option
- Trigger delete action
- Verify confirmation dialog (if any)
- Verify record is removed
- Verify redirect to index

#### 4.7 Validation Testing
- Submit forms with invalid data
- Verify error messages display
- Test required field validation
- Test format validation (email, etc.)

#### 4.8 UX Interaction Testing
- Test dropdown menus
- Test modal dialogs
- Test flash messages
- Test loading states
- Test responsive behavior

### Phase 5: Documentation

1. **Stop GIF Recording**
   ```javascript
   mcp__claude-in-chrome__computer(action="screenshot", tabId=TAB_ID)
   mcp__claude-in-chrome__gif_creator(action="stop_recording", tabId=TAB_ID)
   ```

2. **Export GIF**
   ```javascript
   mcp__claude-in-chrome__gif_creator(
     action="export",
     tabId=TAB_ID,
     filename="feature-test-[feature-name].gif",
     download=true
   )
   ```

3. **Generate Test Report**
   - List all routes tested
   - Note any issues found
   - Include screenshots/GIF reference
   - Summarize CRUD operation results

## Sample Data Generation

When creating test records, use realistic sample data:

### User Data
```
First Name: Test
Last Name: User
Email: test[timestamp]@example.com
Password: password123
```

### Cluster Data
```
Name: Sample Library Network
Description: A test cluster for verifying the application functionality
Locality: Test City
```

### Library Location Data
```
Name: Main Branch Library
Address: 123 Test Street, Test City
Description: Primary library location for testing
```

## Error Handling

If errors are encountered during testing:

1. **Screenshot the Error**
   - Capture the full error message
   - Note the URL and action that caused it

2. **Check Console Messages**
   ```javascript
   mcp__claude-in-chrome__read_console_messages(tabId=TAB_ID, onlyErrors=true)
   ```

3. **Document the Issue**
   - Note which test step failed
   - Include error details in report
   - Suggest potential fixes

## Test Report Template

```markdown
# Feature Test Report: [Feature Name]

**Date:** [Date]
**Tester:** Claude
**Environment:** Development (localhost:PORT)

## Summary
- **Routes Tested:** X of Y
- **CRUD Operations:** [Pass/Fail status for each]
- **Issues Found:** X

## Routes Tested
| Route | Status | Notes |
|-------|--------|-------|
| GET /feature | ✅ Pass | |
| GET /feature/new | ✅ Pass | |
| POST /feature | ✅ Pass | |
| GET /feature/:id | ✅ Pass | |
| GET /feature/:id/edit | ✅ Pass | |
| PATCH /feature/:id | ✅ Pass | |

## CRUD Operations
- **Create:** ✅ Successfully created sample record
- **Read:** ✅ Record displays correctly
- **Update:** ✅ Changes saved successfully
- **Delete:** ✅ Record removed correctly

## Sample Records Created
- [List of test records with IDs]

## Issues Found
- [List any bugs or issues]

## Screenshots/Recording
- GIF: feature-test-[name].gif
```

## Related Commands

- **wheels-debugging**: Use when errors are encountered
- **implementation-verifier**: For more comprehensive verification
- **wheels-test-generator**: For creating automated test specs
