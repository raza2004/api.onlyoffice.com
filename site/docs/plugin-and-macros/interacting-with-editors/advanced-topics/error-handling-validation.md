---
sidebar_position: -1
---

# Error handling & validation

## Overview

Robust error handling and validation are critical for creating reliable ONLYOFFICE plugins. This guide covers strategies for preventing errors, handling failures gracefully, and validating user input to ensure plugin stability.

## Types of errors

### Runtime errors

Errors that occur during plugin execution:

- **Type errors** - Incorrect data types
- **Reference errors** - Undefined variables or functions
- **API errors** - Failed ONLYOFFICE API calls
- **Network errors** - Failed external requests

### Validation errors

Errors from invalid user input:

- **Empty fields** - Required data missing
- **Format errors** - Incorrect data format
- **Range errors** - Values outside acceptable limits
- **Constraint violations** - Business rule violations

### System errors

Errors from external factors:

- **Permission errors** - Insufficient privileges
- **Resource errors** - Memory or storage issues
- **Compatibility errors** - Unsupported features
- **Timeout errors** - Operations taking too long

## Basic error handling

### Try-catch blocks

Wrap risky code in try-catch blocks:

```javascript
function processUserInput(input) {
  try {
    // Risky operations
    const data = JSON.parse(input);
    const result = transform(data);
    updateDocument(result);

    return { success: true, result: result };
  } catch (error) {
    console.error("Processing failed:", error);
    return { success: false, error: error.message };
  }
}
```

### Handling API errors

Wrap ONLYOFFICE API calls with error handling:

```javascript
function insertTextSafely(text) {
  try {
    window.Asc.plugin.executeMethod("PasteText", [text], function (result) {
      if (result === undefined || result === null) {
        throw new Error("Failed to insert text");
      }
      console.log("Text inserted successfully");
    });
  } catch (error) {
    console.error("API call failed:", error);
    showErrorMessage("Could not insert text. Please try again.");
  }
}
```

### Async error handling

Handle errors in async operations:

```javascript
async function fetchAndProcess() {
  try {
    showLoading();

    const response = await fetch("https://api.example.com/data");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    await processData(data);

    hideLoading();
    showSuccess("Data processed successfully");
  } catch (error) {
    hideLoading();

    if (error.name === "TypeError") {
      showError("Network error. Please check your connection.");
    } else {
      showError(`Error: ${error.message}`);
    }

    console.error("Operation failed:", error);
  }
}
```

## Input validation

### Required field validation

```javascript
function validateRequiredFields(data) {
  const errors = [];

  const requiredFields = ["name", "email", "message"];

  requiredFields.forEach((field) => {
    if (!data[field] || data[field].trim() === "") {
      errors.push(`${field} is required`);
    }
  });

  return {
    isValid: errors.length === 0,
    errors: errors,
  };
}

// Usage
function handleSubmit() {
  const formData = {
    name: document.getElementById("name").value,
    email: document.getElementById("email").value,
    message: document.getElementById("message").value,
  };

  const validation = validateRequiredFields(formData);

  if (!validation.isValid) {
    showValidationErrors(validation.errors);
    return;
  }

  processForm(formData);
}
```

### Format validation

```javascript
const validators = {
  email: function (value) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(value);
  },

  url: function (value) {
    try {
      new URL(value);
      return true;
    } catch {
      return false;
    }
  },

  phone: function (value) {
    const phoneRegex = /^\+?[\d\s-()]+$/;
    return phoneRegex.test(value) && value.replace(/\D/g, "").length >= 10;
  },

  number: function (value) {
    return !isNaN(parseFloat(value)) && isFinite(value);
  },
};

function validateField(fieldType, value) {
  if (validators[fieldType]) {
    return validators[fieldType](value);
  }
  return true; // No validator, assume valid
}

// Usage
const email = document.getElementById("email").value;
if (!validateField("email", email)) {
  showError("Please enter a valid email address");
}
```

### Range validation

```javascript
function validateRange(value, min, max, fieldName) {
  const errors = [];

  if (value < min) {
    errors.push(`${fieldName} must be at least ${min}`);
  }

  if (value > max) {
    errors.push(`${fieldName} must not exceed ${max}`);
  }

  return {
    isValid: errors.length === 0,
    errors: errors,
  };
}

// Usage
const fontSize = parseInt(document.getElementById("fontSize").value);
const validation = validateRange(fontSize, 8, 72, "Font size");

if (!validation.isValid) {
  showValidationErrors(validation.errors);
}
```

### Custom validation rules

```javascript
class ValidationRule {
  constructor(name, validator, errorMessage) {
    this.name = name;
    this.validator = validator;
    this.errorMessage = errorMessage;
  }

  validate(value) {
    return {
      isValid: this.validator(value),
      error: this.validator(value) ? null : this.errorMessage,
    };
  }
}

// Define custom rules
const passwordStrength = new ValidationRule(
  "passwordStrength",
  (value) => value.length >= 8 && /[A-Z]/.test(value) && /[0-9]/.test(value),
  "Password must be at least 8 characters with uppercase and numbers",
);

const uniqueUsername = new ValidationRule(
  "uniqueUsername",
  async (value) => {
    const response = await fetch(`/api/check-username?name=${value}`);
    const data = await response.json();
    return data.available;
  },
  "Username is already taken",
);

// Usage
const result = passwordStrength.validate("MyPass123");
if (!result.isValid) {
  showError(result.error);
}
```

## Comprehensive validation framework

### Form validator

```javascript
class FormValidator {
  constructor() {
    this.rules = {};
    this.errors = {};
  }

  addRule(field, rules) {
    this.rules[field] = rules;
    return this;
  }

  validate(data) {
    this.errors = {};
    let isValid = true;

    Object.keys(this.rules).forEach((field) => {
      const fieldRules = this.rules[field];
      const value = data[field];

      fieldRules.forEach((rule) => {
        if (!rule.validator(value)) {
          isValid = false;
          if (!this.errors[field]) {
            this.errors[field] = [];
          }
          this.errors[field].push(rule.message);
        }
      });
    });

    return {
      isValid: isValid,
      errors: this.errors,
    };
  }

  getErrors() {
    return this.errors;
  }
}

// Usage
const validator = new FormValidator();

validator
  .addRule("email", [
    {
      validator: (v) => v && v.length > 0,
      message: "Email is required",
    },
    {
      validator: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
      message: "Please enter a valid email",
    },
  ])
  .addRule("age", [
    {
      validator: (v) => v >= 18,
      message: "Must be at least 18 years old",
    },
    {
      validator: (v) => v <= 120,
      message: "Please enter a valid age",
    },
  ]);

const formData = {
  email: "user@example.com",
  age: 25,
};

const result = validator.validate(formData);
if (!result.isValid) {
  displayErrors(result.errors);
}
```

## Error recovery strategies

### Retry mechanism

```javascript
async function retryOperation(operation, maxRetries = 3, delay = 1000) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`Attempt ${attempt} of ${maxRetries}`);
      const result = await operation();
      return { success: true, result: result };
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${attempt} failed:`, error);

      if (attempt < maxRetries) {
        console.log(`Retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  return {
    success: false,
    error: lastError,
    message: `Failed after ${maxRetries} attempts`,
  };
}

// Usage
async function saveData(data) {
  const result = await retryOperation(
    () =>
      fetch("/api/save", {
        method: "POST",
        body: JSON.stringify(data),
      }),
    3,
    2000,
  );

  if (!result.success) {
    showError("Failed to save data after multiple attempts");
  }
}
```

### Fallback values

```javascript
function getConfigValue(key, fallback) {
  try {
    const config = JSON.parse(localStorage.getItem("pluginConfig"));
    return config[key] !== undefined ? config[key] : fallback;
  } catch (error) {
    console.error("Failed to load config:", error);
    return fallback;
  }
}

// Usage
const fontSize = getConfigValue("fontSize", 12);
const theme = getConfigValue("theme", "light");
```

### Graceful degradation

```javascript
function insertContentWithFallback(content) {
  // Try advanced method first
  try {
    window.Asc.plugin.executeMethod("PasteHtml", [content]);
  } catch (error) {
    console.warn("HTML insertion failed, using plain text");

    // Fallback to plain text
    try {
      const plainText = stripHtml(content);
      window.Asc.plugin.executeMethod("PasteText", [plainText]);
    } catch (fallbackError) {
      console.error("All insertion methods failed");
      showError("Could not insert content");
    }
  }
}
```

## User-friendly error messages

### Error message mapping

```javascript
const errorMessages = {
  NETWORK_ERROR: "Unable to connect. Please check your internet connection.",
  AUTH_ERROR: "Authentication failed. Please sign in again.",
  PERMISSION_ERROR: "You do not have permission to perform this action.",
  VALIDATION_ERROR: "Please check your input and try again.",
  TIMEOUT_ERROR: "The operation took too long. Please try again.",
  UNKNOWN_ERROR: "An unexpected error occurred. Please try again.",
};

function getUserFriendlyError(error) {
  if (error.code && errorMessages[error.code]) {
    return errorMessages[error.code];
  }

  // Check error message patterns
  if (error.message.includes("network")) {
    return errorMessages.NETWORK_ERROR;
  }
  if (error.message.includes("timeout")) {
    return errorMessages.TIMEOUT_ERROR;
  }

  return errorMessages.UNKNOWN_ERROR;
}

// Usage
try {
  await performOperation();
} catch (error) {
  const friendlyMessage = getUserFriendlyError(error);
  showError(friendlyMessage);

  // Log technical details for debugging
  console.error("Technical error:", error);
}
```

### Error notification UI

```javascript
function showErrorNotification(message, options = {}) {
  const notification = document.createElement("div");
  notification.className = "error-notification";
  notification.innerHTML = `
    <div class="error-icon">⚠️</div>
    <div class="error-content">
      <div class="error-title">${options.title || "Error"}</div>
      <div class="error-message">${message}</div>
      ${options.details ? `<div class="error-details">${options.details}</div>` : ""}
    </div>
    <button class="error-close">×</button>
  `;

  notification.querySelector(".error-close").addEventListener("click", () => {
    notification.remove();
  });

  document.body.appendChild(notification);

  // Auto-remove after timeout
  if (options.timeout !== false) {
    setTimeout(() => {
      notification.remove();
    }, options.timeout || 5000);
  }
}

// Usage
showErrorNotification("Failed to save your changes", {
  title: "Save Error",
  details: "Please try again or contact support if the problem persists",
  timeout: 7000,
});
```

## Logging and debugging

### Error logging service

```javascript
class ErrorLogger {
  constructor() {
    this.errors = [];
    this.maxErrors = 100;
  }

  log(error, context = {}) {
    const errorEntry = {
      timestamp: new Date().toISOString(),
      message: error.message,
      stack: error.stack,
      context: context,
      userAgent: navigator.userAgent,
    };

    this.errors.push(errorEntry);

    // Keep only recent errors
    if (this.errors.length > this.maxErrors) {
      this.errors.shift();
    }

    // Log to console in development
    if (isDevelopment()) {
      console.error("Error logged:", errorEntry);
    }

    // Send to server in production
    if (isProduction()) {
      this.sendToServer(errorEntry);
    }
  }

  async sendToServer(errorEntry) {
    try {
      await fetch("/api/log-error", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(errorEntry),
      });
    } catch (error) {
      console.error("Failed to send error log:", error);
    }
  }

  getErrors() {
    return this.errors;
  }

  clearErrors() {
    this.errors = [];
  }
}

// Global error logger
const errorLogger = new ErrorLogger();

// Usage
try {
  performRiskyOperation();
} catch (error) {
  errorLogger.log(error, {
    operation: "performRiskyOperation",
    user: getCurrentUser(),
    data: operationData,
  });
}
```

## Best practices

### Validate early

```javascript
// ✅ Good - Validate before processing
function processData(data) {
  if (!data || typeof data !== 'object') {
    throw new Error('Invalid data: object expected');
  }

  if (!data.required Field) {
    throw new Error('Missing required field');
  }

  // Proceed with processing
  return transform(data);
}

// ❌ Bad - Processing then discovering errors
function processData(data) {
  const step1 = transform(data); // May fail deep in processing
  const step2 = validate(step1); // Too late
  return step2;
}
```

### Provide actionable errors

```javascript
// ❌ Bad - Vague error
throw new Error("Invalid input");

// ✅ Good - Specific and actionable
throw new Error("Email format is invalid. Please use format: user@example.com");
```

### Don't swallow errors silently

```javascript
// ❌ Bad - Silent failure
try {
  riskyOperation();
} catch (error) {
  // Nothing - user doesn't know it failed
}

// ✅ Good - Log and notify
try {
  riskyOperation();
} catch (error) {
  console.error("Operation failed:", error);
  showError("Unable to complete operation");
}
```

## Conclusion

Implementing comprehensive error handling and validation ensures your ONLYOFFICE plugin remains stable and provides a professional user experience. By anticipating failures, validating inputs, and handling errors gracefully, you create plugins that users can rely on.
