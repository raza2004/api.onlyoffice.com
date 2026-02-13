---
sidebar_position: 1
---

# Getting started with ONLYOFFICE plugins

## Introduction

ONLYOFFICE is a powerful open-source office suite that supports documents, spreadsheets, and presentations. One of its most valuable features is the ability to extend functionality through plugins. This guide will help you get started with developing your own ONLYOFFICE plugins.

## Prerequisites

Before you begin developing ONLYOFFICE plugins, you should have:

- Basic knowledge of JavaScript, HTML, and CSS
- A text editor or IDE for development
- ONLYOFFICE Desktop Editors or Document Server installed
- Understanding of JSON configuration files

## Setting up your development environment

To start developing ONLYOFFICE plugins, follow these steps:

1. **Install ONLYOFFICE**: Download and install ONLYOFFICE Desktop Editors from the official website or set up ONLYOFFICE Document Server.

2. **Locate the plugins directory**: Find where ONLYOFFICE stores its plugins on your system:
   - **Windows**: `%ProgramFiles%\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\`
   - **Linux**: `/opt/ONLYOFFICE/desktopeditors/editors/sdkjs-plugins/`
   - **macOS**: `/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/`

3. **Create your plugin folder**: Create a new folder in the plugins directory with a descriptive name for your plugin.

## Basic plugin structure

Every ONLYOFFICE plugin requires at least these essential files:

```
my-plugin/
├── config.json
├── index.html
├── pluginCode.js
└── icon.png
```

### Minimum configuration

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

## Your first plugin

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
    <h1>Hello ONLYOFFICE!</h1>
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

## Testing your plugin

1. Place your plugin folder in the ONLYOFFICE plugins directory
2. Restart ONLYOFFICE
3. Open a document and look for your plugin in the Plugins tab
4. Click on your plugin to launch it

## Next steps

Now that you have a basic plugin running, you can:

- Explore the ONLYOFFICE API documentation
- Learn about plugin configuration options
- Understand the plugin lifecycle
- Add more sophisticated functionality to your plugins

Ready to dive deeper? Continue to the next section to learn about what plugins are and how they work within ONLYOFFICE.
