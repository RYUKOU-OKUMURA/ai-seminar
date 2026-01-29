# Google Apps Script Troubleshooting Guide

> **Version**: 1.0.0
> **Last Updated**: 2026-01-29
> **Author**: Research Team

## Table of Contents

1. [Error Handling](#error-handling)
2. [Logging](#logging)
3. [Debugging Techniques](#debugging-techniques)
4. [Common Errors and Solutions](#common-errors-and-solutions)
5. [Error Notifications](#error-notifications)
6. [Testing Environment](#testing-environment)

---

## Error Handling

### try-catch Statement

Google Apps Script supports standard JavaScript error handling using `try-catch-finally`:

```javascript
function processData() {
  try {
    // Code that might throw an error
    var sheet = SpreadsheetApp.getActiveSheet();
    var data = sheet.getDataRange().getValues();

    // Process data...
  } catch (error) {
    // Handle the error
    Logger.log('Error occurred: ' + error.toString());
    MailApp.sendEmail({
      to: 'admin@example.com',
      subject: 'Script Error',
      body: 'An error occurred: ' + error.toString()
    });
  } finally {
    // Cleanup code (always executes)
    Logger.log('Process completed');
  }
}
```

### Exception Types

Google Apps Script uses standard JavaScript error types:

| Error Type | Description | Example |
|------------|-------------|---------|
| `Error` | Base error type | `throw new Error('Generic error')` |
| `TypeError` | Type mismatch | `null.method()` |
| `ReferenceError` | Invalid reference | `nonExistentVar` |
| `RangeError` | Numeric value out of range | `Array(-1)` |
| `SyntaxError` | Invalid JavaScript syntax | Caught on save |

### Apps Script-Specific Errors

```javascript
// Common Apps Script exceptions
try {
  MailApp.sendEmail('invalid-email', 'Subject', 'Body');
} catch (e) {
  // e.message contains details like "Invalid email: invalid-email"
  console.error('MailApp Error:', e.message);
}
```

### Error Handling Patterns

#### 1. Loop Processing with Continuation

```javascript
function processItems(items) {
  var results = [];
  var errors = [];

  for (var i = 0; i < items.length; i++) {
    try {
      var result = processItem(items[i]);
      results.push(result);
    } catch (error) {
      errors.push({
        item: items[i],
        error: error.toString()
      });
      // Continue processing next item
    }
  }

  return { results: results, errors: errors };
}
```

#### 2. Web App Endpoint Error Handling

```javascript
function doGet(e) {
  try {
    var data = fetchData();
    return ContentService.createTextOutput(JSON.stringify(data))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      error: error.toString(),
      timestamp: new Date()
    }))
    .setMimeType(ContentService.MimeType.JSON);
  }
}
```

#### 3. API Call Retry Pattern

```javascript
function fetchWithRetry(url, maxRetries) {
  maxRetries = maxRetries || 3;

  for (var i = 0; i < maxRetries; i++) {
    try {
      return UrlFetchApp.fetch(url);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      Utilities.sleep(1000 * (i + 1)); // Exponential backoff
    }
  }
}
```

---

## Logging

### Logger.log()

The `Logger.log()` service writes to both:
- **Execution Log** (visible in Apps Script editor)
- **Google Cloud Logging** (formerly Stackdriver Logging)

```javascript
function logExample() {
  Logger.log('Simple message');

  var value = 42;
  Logger.log('Value: %s', value);

  var obj = { name: 'Test', count: 10 };
  Logger.log('Object: %s', JSON.stringify(obj));
}
```

**To view logs**: In the Apps Script editor, click **Execution log** after running a function.

### Console Logging

Modern Apps Script supports standard `console` methods that write to Cloud Logging:

```javascript
function consoleExample() {
  console.log('DEBUG message');     // DEBUG level
  console.info('INFO message');      // INFO level
  console.warn('WARNING message');   // WARNING level
  console.error('ERROR message');    // ERROR level
}
```

### Cloud Logging Integration

All logs are automatically sent to Google Cloud Logging. To access:

1. In Apps Script editor: **View > Stackdriver Logging** (now called **Cloud Logging**)
2. Or directly in [Google Cloud Console](https://console.cloud.google.com/logs)

#### Structured Logging

```javascript
function structuredLogging() {
  console.log({
    message: 'User action',
    userId: Session.getActiveUser().getEmail(),
    action: 'create',
    timestamp: new Date().toISOString()
  });
}
```

### Log Severity Levels

| Method | Level | Use Case |
|--------|-------|----------|
| `console.log()` | DEBUG | Development information |
| `console.info()` | INFO | General information |
| `console.warn()` | WARNING | Non-critical issues |
| `console.error()` | ERROR | Errors and exceptions |

---

## Debugging Techniques

### Using the Debugger

#### Setting Breakpoints

1. Click the **line number** gutter in the Apps Script editor
2. A red circle indicates a breakpoint is set

#### Running in Debug Mode

1. Click the **Debug** button in the toolbar
2. Script pauses at breakpoints
3. Inspect variables in the **Debugger** panel

#### Debugger Controls

| Control | Action | Keyboard Shortcut |
|---------|--------|-------------------|
| **Step Over** | Execute current line, skip function calls | F10 |
| **Step Into** | Execute current line, enter function calls | F11 |
| **Step Out** | Exit current function | Shift+F11 |
| **Resume** | Continue execution | F5 |

### Variable Inspection

When paused at a breakpoint:
- **Local variables**: Displayed in the Debugger panel
- **Watch expressions**: Add custom expressions to monitor
- **Call stack**: View the execution path

### Viewing Executions

Every script run is recorded. To view:

1. Click **Executions** in the left sidebar
2. View execution history including:
   - Function name
   - Start time
   - Duration
   - Status (success/error)
   - Cloud Logs link

### Error Reporting

Runtime exceptions are automatically captured by **Google Cloud Error Reporting**:

1. Go to [Google Cloud Console > Error Reporting](https://console.cloud.google.com/errors)
2. Filter by your Apps Script project
3. View grouped exception messages with stack traces

### Debugging Dynamic Code

**Note**: Dynamically generated JavaScript (via `eval()`, `new Function()`) cannot be displayed in the debugger.

```javascript
function debugDynamicCode() {
  eval('var x = 2;');  // Cannot step into this code
  // Error: "Source code for the current line is not available"
}
```

**Workaround**: Use `//# sourceURL` comment to name dynamic scripts:

```javascript
eval('var x = 2;\n//# sourceURL=myDynamicScript.js');
```

---

## Common Errors and Solutions

### Syntax Errors

**Detected on save** - File cannot be saved until fixed.

```javascript
// WRONG: Missing closing parenthesis
var text = data.join(" ";

// CORRECT
var text = data.join(" ");
```

### Runtime Errors

**Detected during execution**.

#### Invalid Email

```
Error: Invalid email: john (line 5)
```

**Solution**: Validate email addresses before sending:

```javascript
function sendEmail(address) {
  if (!address || !address.includes('@')) {
    throw new Error('Invalid email address: ' + address);
  }
  MailApp.sendEmail(address, 'Subject', 'Body');
}
```

#### Quota Exceeded

```
Error: Service invoked too many times: MailApp
```

**Solution**:
- Check [Apps Script Quotas](https://developers.google.com/apps-script/guides/services/quotas)
- Implement rate limiting or batching

#### Service Not Available

```
Error: Server not available
```

**Solutions**:
1. Wait a moment and retry (temporary server issue)
2. Check for script bugs without error messages
3. Report as a bug if persistent

#### Authorization Required

```
Error: Script is not authorized
```

**Solution**: Open script in editor and run any function to authorize.

#### Drive API Disabled

```
Error: The domain policy has disabled third-party Drive apps
```

**Cause**: Google Workspace admin has disabled Drive API.

**Solution**: Contact administrator or deploy as domain-wide install.

#### User Identity Not Available

```
Error: The script does not have permission to get the active user's identity
```

**Solutions by AuthMode**:

| AuthMode | Solution |
|----------|----------|
| `AuthMode.FULL` | Use `Session.getEffectiveUser()` instead |
| `AuthMode.LIMITED` | Ensure owner has authorized |
| Other modes | Avoid calling `getActiveUser()` |

#### Library Missing

```
Error: Library is missing
```

**Solutions**:
1. Copy library code directly into your script
2. Create a copy of the library from your account
3. Use a different version of the library

#### Library Version Not Found

```
Error: Error code Not_Found - missing library version
```

**Solutions**:
1. Update library to existing version
2. Remove deleted library from project
3. Check for transitive library dependencies

### Multiple Account Issues

**Problem**: Being logged into multiple Google Accounts simultaneously is not supported.

**Solutions**:
1. Log out of all accounts, log in with only one
2. Use incognito/private browsing window
3. Use separate browser profiles

### Service Outages

Check [Google Workspace Status Dashboard](https://www.google.com/appsstatus) for service issues.

---

## Error Notifications

### Automatic Failure Notifications

Google Apps Script automatically sends "Summary of failures" emails when:

- Time-driven triggers fail
- Installable triggers fail to execute
- Functions run via triggers encounter errors

**Note**: Add-on triggers do NOT send these notifications.

### Managing Failure Notifications

1. **Configure via link** in failure email
2. Access at [script.google.com](https://script.google.com)
3. Modify trigger notification preferences

### Custom Error Notifications

```javascript
function customErrorHandler(error) {
  var message = [
    'Script Error Report',
    '===================',
    'Error: ' + error.toString(),
    'Time: ' + new Date(),
    'Script: ' + ScriptApp.getScriptId(),
    'User: ' + Session.getActiveUser().getEmail()
  ].join('\n');

  MailApp.sendEmail({
    to: 'admin@example.com',
    subject: '🚨 Script Error Alert',
    body: message
  });
}
```

### Test Environment Validation

Use built-in testing for validation:

```javascript
function testFunction() {
  // Test mode detection
  var isTest = ScriptApp.getProjectTriggers().length === 0;

  if (isTest) {
    console.log('Running in test mode');
    // Use test data instead of production
  }
}
```

---

## Testing Environment

### Manual Testing

1. **Run individual functions** from the editor
2. **Use test data** instead of production data
3. **Check Execution log** after each run

### Debugging Triggers

Triggers can be challenging to debug:

1. Add `Logger.log()` statements
2. Check Cloud Logging for output
3. Use try-catch to email errors to yourself

```javascript
function myTrigger() {
  try {
    // Your code here
    Logger.log('Trigger executed successfully');
  } catch (error) {
    Logger.log('Trigger error: ' + error.toString());
    MailApp.sendEmail('your-email@example.com', 'Trigger Error', error.toString());
  }
}
```

### Best Practices

1. **Always use try-catch** in production code
2. **Log strategically** - avoid excessive logging
3. **Test with realistic data** - edge cases matter
4. **Monitor quotas** - especially for automated scripts
5. **Set up error notifications** - be proactive about issues

---

## References

- [Apps Script Troubleshooting Guide](https://developers.google.com/apps-script/guides/support/troubleshooting)
- [Apps Script Logging](https://developers.google.com/apps-script/guides/logging)
- [Apps Script Quotas](https://developers.google.com/apps-script/guides/services/quotas)
- [Google Workspace Status Dashboard](https://www.google.com/appsstatus)
- [Cloud Logging Logs Explorer](https://console.cloud.google.com/logs)
