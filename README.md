# Hubstaff API Support Hub

A browser-based support tool for quickly identifying and resolving Hubstaff API errors from customer messages.

## What It Does

This tool helps support agents diagnose API errors by:

1. **Analyzing customer messages** - Paste an email or Slack message containing an API error, and the tool automatically extracts error codes, keys, HTTP status codes, and context
2. **Matching to known errors** - Cross-references extracted data against all 47 Hubstaff API error codes
3. **Providing support guidance** - Shows step-by-step troubleshooting instructions with copyable response templates
4. **Quick Reference** - Common errors, HTTP status codes, and ready-to-use response templates

---

## ⚠️ Important: Error Code Coverage

**Current state:** Structured error codes (with `error_code` numbers) are currently only returned by **time entry creation endpoints**.

**General errors:** Auth/permission errors (10xxx) and resource errors (12xxx like "not_found") work across **all API endpoints**.

**Future:** Engineering is working to add structured error codes to all Public API endpoints.

---

## Features

### 🔍 Message Analyzer Tab

The main workspace for diagnosing customer issues.

**How to use:**
1. Paste the customer's message (email, Slack, ticket) into the text area
2. Click **Analyze Message**
3. Review the extracted information and matched errors
4. Copy the response template if provided

**What it extracts:**
- `error_code` - Numeric error code (e.g., 11000)
- `code` - String error key (e.g., "invalid_params")
- `error` - Error message text
- HTTP status codes (400, 401, 403, 404, 429, 500, etc.)
- API endpoints mentioned (`/v2/...`)
- Context keywords (task, project, integration, Jira, etc.)

**Match Types:**
- **Exact** (green border) - Error code or key found directly in the message
- **Related** (blue badge) - Suggested based on context keywords; verify with customer

### 📚 Error Browser Tab

Browse and search all 47 Hubstaff API error codes.

**Features:**
- Search by code, key, or message text
- Filter by category (Auth, Validation, Resource, Rate Limit, Time/Activity, System)
- Click any error to view full details in a modal
- Green checkmark indicates errors with support guidance

### ⚡ Quick Reference Tab

Fast access to common information:

- **Most Common Errors** - The 6 most frequently encountered errors with guidance
- **HTTP Status Quick Guide** - What each status code means in plain English
- **Response Templates** - Pre-written customer responses ready to copy

### 📊 Usage Statistics

Tracks which errors you search most frequently (stored in browser localStorage).

- Shows top 5 most-searched errors
- Helps identify common customer issues
- Clear statistics button to reset

### 📖 User Guide

Built-in help system with:

- **Quick Start** - 30-second guide to using the tool
- **Analyzer** - Detailed explanation of message analysis
- **Categories** - Error code ranges and what they mean
- **Glossary** - API terms explained in plain English
- **Pro Tips** - Expert advice for handling API support

---

## Error Categories

| Range | Category | Description |
|-------|----------|-------------|
| 10xxx | Authentication | Token permissions, authorization issues |
| 11xxx | Validation | Invalid parameters, duplicate records |
| 12xxx | Resource | Not found, access denied |
| 13xxx | Rate Limit | Too many requests (350/minute limit) |
| 14xxx | Time/Activity | Time entries, activities, timesheets |
| 15xxx | System | Server errors, timeouts |

---

## Support Guidance

The tool includes built-in troubleshooting guidance for these common errors:

| Code | Key | Has Guidance | Has Template |
|------|-----|--------------|--------------|
| 10001 | auth_not_authorized | ✅ | ✅ |
| 10003 | auth_forbidden | ✅ | ✅ |
| 10004 | auth_financials_forbidden | ✅ | ✅ |
| 11000 | validation_invalid_params | ✅ | ✅ |
| 11002 | validation_record_not_unique | ✅ | ✅ |
| 11003 | validation_lock_version | ✅ | ✅ |
| 12000 | resource_not_found | ✅ | ✅ |
| 12002 | resource_user_project_access | ✅ | ✅ |
| 13000 | rate_limit_exceeded | ✅ | ✅ |
| 14001 | time_entry_approval_required | ✅ | ✅ |
| 14101 | activity_locked | ✅ | ✅ |
| 14102 | activity_period_restricted | ✅ | ✅ |
| 14403 | activity_timesheet_locked | ✅ | ✅ |
| 15000 | system_unknown_error | ✅ | ✅ |
| 15001 | system_service_unavailable | ✅ | ✅ |

Errors without guidance show a yellow alert.

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
        ],
        template: "This error occurs when another update is already in progress for the same resource. Please wait a few seconds and try your request again."
    }
};
```

**Fields:**
- `title` - Short name for the error type
- `guidance` - One-sentence explanation for support agents
- `steps` - Array of troubleshooting steps (shown as bullet points)
- `template` - Pre-written customer response (optional, enables "Copy" button)

---

## Technical Details

### Data Source

Error codes come from the official Hubstaff API endpoint:

```
GET https://api.hubstaff.com/v2/error_codes
```

Documentation: [developer.hubstaff.com/docs/hubstaff_v2#tag/errors](https://developer.hubstaff.com/docs/hubstaff_v2#tag/errors)

The current database contains 47 error codes.

### Storage

- Usage statistics stored in `localStorage` under key `hubstaffErrorStats`
- No server-side storage; everything runs in the browser

### File Structure

```
hubstaff-api-support-hub.html  - Complete tool (single file, all-in-one)
README.md                       - This documentation
```

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
6. Click **Copy** on the response template

**Response to customer:**
> When a project is connected to an external integration (like Jira or Redmine), tasks must be created in that external system rather than through the Hubstaff API. The integration will automatically sync tasks to Hubstaff. This is by design to maintain data consistency between systems.

---

## Browser Support

Works in all modern browsers:
- Chrome / Edge (recommended)
- Firefox
- Safari

No installation required - just open the HTML file.

---

## Useful Links

- **API Documentation:** [developer.hubstaff.com/docs/hubstaff_v2](https://developer.hubstaff.com/docs/hubstaff_v2)
- **Authentication Guide:** [developer.hubstaff.com/authentication](https://developer.hubstaff.com/authentication)
- **Error Codes Reference:** [developer.hubstaff.com/docs/hubstaff_v2#tag/errors](https://developer.hubstaff.com/docs/hubstaff_v2#tag/errors)
