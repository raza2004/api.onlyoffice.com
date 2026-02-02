---
sidebar_position: 1
---

# Getting started with OnlyOffice plugins

## Introduction

OnlyOffice is a powerful open-source office suite that supports documents, spreadsheets, and presentations. One of its most valuable features is the ability to extend functionality through plugins. This guide will help you get started with developing your own OnlyOffice plugins.

## Prerequisites

Before you begin developing OnlyOffice plugins, you should have:

- Basic knowledge of JavaScript, HTML, and CSS
- A text editor or IDE for development
- OnlyOffice Desktop Editors or Document Server installed
- Understanding of JSON configuration files

## Setting Up Your Development Environment

To start developing OnlyOffice plugins, follow these steps:

1. **Install OnlyOffice**: Download and install OnlyOffice Desktop Editors from the official website or set up OnlyOffice Document Server.

2. **Locate the Plugins Directory**: Find where OnlyOffice stores its plugins on your system:
   - **Windows**: `%ProgramFiles%\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\`
   - **Linux**: `/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/`
   - **macOS**: `/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/`

3. **Create Your Plugin Folder**: Create a new folder in the plugins directory with a descriptive name for your plugin.

## Basic Plugin Structure

Every OnlyOffice plugin requires at least these essential files:

```
my-plugin/
├── config.json
├── index.html
├── pluginCode.js
└── icon.png
```

### Minimum Configuration

Create a `config.json` file with the basic plugin information:

```json
{
  "name": "My First Plugin",
  "guid": "asc.{YOUR-UNIQUE-GUID}",
  "version": "1.0.0",
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png"],
      "isViewer": false,
      "EditorsSupport": ["word", "cell", "slide"]
    }
  ]
}
```

## Your First Plugin

Let's create a simple "Hello World" plugin:

**index.html**:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello Plugin</title>
    <script src="pluginCode.js"></script>
  </head>
  <body>
    <h1>Hello OnlyOffice!</h1>
    <button onclick="insertText()">Insert Text</button>
  </body>
</html>
```

**pluginCode.js**:

```javascript
(function (window, undefined) {
  window.insertText = function () {
    window.Asc.plugin.executeMethod("PasteText", ["Hello from my plugin!"]);
  };

  window.Asc.plugin.init = function () {
    console.log("Plugin initialized");
  };
})(window, undefined);
```

## Testing Your Plugin

1. Place your plugin folder in the OnlyOffice plugins directory
2. Restart OnlyOffice
3. Open a document and look for your plugin in the Plugins tab
4. Click on your plugin to launch it

## Next Steps

Now that you have a basic plugin running, you can:

- Explore the OnlyOffice API documentation
- Learn about plugin configuration options
- Understand the plugin lifecycle
- Add more sophisticated functionality to your plugins

Ready to dive deeper? Continue to the next section to learn about what plugins are and how they work within OnlyOffice.
