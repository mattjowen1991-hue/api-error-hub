# Hubstaff API Error Diagnostic Tool

A browser-based support tool for quickly identifying and resolving Hubstaff API errors from customer messages.

## What It Does

This tool helps support agents diagnose API errors by:

1. **Analyzing customer messages** - Paste an email or Slack message containing an API error, and the tool automatically extracts error codes, keys, HTTP status codes, and context
2. **Matching to known errors** - Cross-references extracted data against all 47 Hubstaff API error codes
3. **Providing support guidance** - Shows step-by-step troubleshooting instructions for common errors
4. **Linking to API Hub** - One-click search in the API Support Assistant for related endpoints

---

## Features

### 📋 Message Analyzer Tab

The main workspace for diagnosing customer issues.

**How to use:**
1. Paste the customer's message (email, Slack, ticket) into the text area
2. Click **Analyze Message**
3. Review the extracted information and matched errors

**What it extracts:**
- `error_code` - Numeric error code (e.g., 11000)
- `code` - String error key (e.g., "invalid_params")
- `error` - Error message text
- HTTP status codes (400, 401, 403, 404, 429, 500, etc.)
- API endpoints mentioned (`/v2/...`)
- Context keywords (task, project, integration, Jira, etc.)

**Match Types:**
- **Exact Match** (green border) - Error code or key found directly in the message
- **Related** (blue badge) - Suggested based on context keywords; verify with customer

### 📚 Error Browser Tab

Browse and search all 47 Hubstaff API error codes.

**Features:**
- Search by code, key, or message text
- Filter by category (Auth, Validation, Resource, Rate Limit, Time/Activity, System)
- Click any error to view full details in a modal

### 📊 Usage Statistics

Tracks which errors you search most frequently (stored in browser localStorage).

- Shows top 5 most-searched errors
- Helps identify common customer issues
- Clear statistics button to reset

### 🔗 API Hub Integration

Each matched error has a "Search API Hub" button that opens the API Support Assistant with a pre-filled search query.

---

## Error Categories

| Range | Category | Description |
|-------|----------|-------------|
| 10xxx | Authentication | Token permissions, authorization issues |
| 11xxx | Validation | Invalid parameters, duplicate records |
| 12xxx | Resource | Not found, access denied |
| 13xxx | Rate Limit | Too many requests |
| 14xxx | Time/Activity | Time entries, activities, timesheets |
| 15xxx | System | Server errors, timeouts |

---

## Support Guidance

The tool includes built-in troubleshooting guidance for these common errors:

| Code | Key | Has Guidance |
|------|-----|--------------|
| 10001 | auth_not_authorized | ✅ |
| 10003 | auth_forbidden | ✅ |
| 10004 | auth_financials_forbidden | ✅ |
| 11000 | validation_invalid_params | ✅ |
| 11002 | validation_record_not_unique | ✅ |
| 11003 | validation_lock_version | ✅ |
| 12000 | resource_not_found | ✅ |
| 12002 | resource_user_project_access | ✅ |
| 13000 | rate_limit_exceeded | ✅ |
| 14001 | time_entry_approval_required | ✅ |
| 14101 | activity_locked | ✅ |
| 14102 | activity_period_restricted | ✅ |
| 14403 | activity_timesheet_locked | ✅ |
| 15000 | system_unknown_error | ✅ |
| 15001 | system_service_unavailable | ✅ |

Errors without guidance show a yellow alert prompting you to add one.

---

## Adding New Guidance

To add support guidance for an error, edit the `supportGuidance` object in the HTML file:

```javascript
const supportGuidance = {
    // ... existing entries ...
    
    12001: {
        title: "Update In Progress",
        guidance: "Another update is currently being processed for this resource.",
        steps: [
            "Wait a few seconds and retry the request",
            "Check if another process or user is updating the same resource",
            "Implement retry logic with exponential backoff"
        ]
    }
};
```

**Fields:**
- `title` - Short name for the error type
- `guidance` - One-sentence explanation for support agents
- `steps` - Array of troubleshooting steps (shown as bullet points)

---

## Technical Details

### Data Source
Error codes are loaded from the Hubstaff API endpoint:
```
GET https://api.hubstaff.com/v2/error_codes
```

The current database contains 47 error codes (as of the last update).

### Storage
- Usage statistics stored in `localStorage` under key `errorUsageStats`
- No server-side storage; everything runs in the browser

### File Structure
```
hubstaff-error-diagnostic-tool.html  - Main tool (single file)
index.html                            - API Support Assistant (linked)
```

### Cross-Tool Integration
The "Search API Hub" button constructs a URL:
```
index.html?search=validation+invalid+params
```

The API Hub reads this parameter and auto-executes the search.

---

## Example Workflow

**Customer email:**
> We're trying to create tasks via the API for a project linked to Redmine, but we get this error:
> ```json
> {
>   "code": "invalid_params",
>   "error_code": 11000,
>   "error": "Can not create a task for a project with a task integration"
> }
> ```

**Steps:**
1. Paste the email into the Message Analyzer
2. Click **Analyze Message**
3. Tool extracts: `error_code: 11000`, `code: invalid_params`, context: `task, project, integration, Redmine`
4. **Exact Match** found: Error 11000 (validation_invalid_params)
5. Guidance shows: "Tasks MUST be created in the source system (Redmine), not via the API"
6. Click **Search API Hub** to find related task endpoints

**Response to customer:**
> When a project is linked to an external integration like Redmine, tasks must be created in Redmine rather than through the Hubstaff API. The integration will automatically sync tasks from Redmine to Hubstaff. This is by design to maintain data consistency between systems.

---

## Browser Support

Works in all modern browsers:
- Chrome / Edge (recommended)
- Firefox
- Safari

No installation required - just open the HTML file.
