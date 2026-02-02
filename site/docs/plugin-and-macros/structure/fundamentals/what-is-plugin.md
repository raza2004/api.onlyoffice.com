---
sidebar_position: 2
---

# What is an OnlyOffice plugin?

## Overview

An OnlyOffice plugin is a modular extension that adds new functionality to OnlyOffice editors (Document Editor, Spreadsheet Editor, and Presentation Editor). Plugins are built using web technologies like HTML, CSS, and JavaScript, making them accessible to web developers.

## Core Concepts

### Plugin Architecture

OnlyOffice plugins operate within a sandboxed iframe environment, communicating with the main editor through a well-defined API. This architecture ensures:

- **Security**: Plugins run in isolation from the core application
- **Compatibility**: Plugins work across different platforms and editor types
- **Flexibility**: Developers can use familiar web technologies

### How Plugins Work

Plugins interact with OnlyOffice through the `window.Asc.plugin` interface, which provides:

1. **Methods to modify documents**: Insert text, images, tables, and other content
2. **Access to document data**: Read current selections, document properties, and metadata
3. **UI integration**: Display custom interfaces within the editor
4. **Event handling**: Respond to editor events and user actions

## Types of Plugins

OnlyOffice supports several categories of plugins:

### 1. Content Manipulation Plugins

These plugins modify document content directly, such as:

- Text formatters and converters
- Code highlighters
- Translation tools
- Content generators

### 2. Integration Plugins

These connect OnlyOffice to external services:

- Cloud storage integrations
- CRM systems
- Project management tools
- Communication platforms

### 3. Enhancement Plugins

These add new capabilities to the editor:

- Advanced formatting tools
- Mathematical equation editors
- Chart and diagram creators
- Image editors

### 4. Utility Plugins

These provide helpful tools for productivity:

- Word counters
- Document statistics
- Template libraries
- Workflow automation

## Plugin Components

### Configuration File (config.json)

The `config.json` file defines the plugin's metadata and behavior:

```json
{
  "name": "Plugin Name",
  "guid": "asc.{unique-identifier}",
  "version": "1.0.0",
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png", "icon@2x.png"],
      "isViewer": false,
      "EditorsSupport": ["word", "cell", "slide"],
      "isVisual": true,
      "isModal": false,
      "isInsideMode": false,
      "initDataType": "none",
      "initData": "",
      "isUpdateOleOnResize": false,
      "buttons": []
    }
  ]
}
```

### User Interface (HTML/CSS)

The plugin's visual interface is built with standard HTML and CSS. The UI can be:

- **Modal**: A popup dialog that requires user interaction
- **Panel**: A side panel that stays open while working
- **Background**: No visible UI, runs silently
- **Inside mode**: Embedded directly in the document

### Business Logic (JavaScript)

JavaScript code handles the plugin's functionality using the OnlyOffice API:

```javascript
(function (window, undefined) {
  // Initialize plugin
  window.Asc.plugin.init = function (data) {
    // Setup code here
  };

  // Handle button clicks
  window.Asc.plugin.button = function (id) {
    // Button handler code
  };

  // Execute methods
  window.Asc.plugin.executeMethod("MethodName", [parameters]);
})(window, undefined);
```

## Plugin Capabilities

### Document Manipulation

Plugins can perform various document operations:

- Insert and format text
- Add tables, images, and shapes
- Modify document styles
- Create and edit macros
- Access and change document properties

### Data Exchange

Plugins can exchange data with:

- External APIs via AJAX requests
- Local storage for persistence
- The editor's internal document model
- Other plugins (in some cases)

### User Interaction

Plugins can create rich user experiences:

- Custom forms and input fields
- Interactive dialogs and wizards
- Real-time previews
- Drag-and-drop interfaces

## Plugin Distribution

OnlyOffice plugins can be distributed in several ways:

1. **Plugin Marketplace**: Submit to the official OnlyOffice store
2. **Manual Installation**: Users copy plugin files to their plugins directory
3. **Server-side Deployment**: Administrators install plugins on Document Server
4. **GitHub/Repository**: Share open-source plugins with the community

## Benefits of Using Plugins

### For Users

- Extend editor functionality without switching applications
- Customize workflows to specific needs
- Integrate favorite tools and services
- Automate repetitive tasks

### For Developers

- Leverage existing web development skills
- Access a growing user base
- Create solutions for specific industries or use cases
- Monetize plugins through the marketplace

## Security Considerations

When developing plugins, keep these security practices in mind:

- Validate all user inputs
- Use HTTPS for external API calls
- Don't store sensitive data in plain text
- Follow the principle of least privilege
- Sanitize content before inserting into documents

## Plugin Limitations

While powerful, plugins have certain constraints:

- Limited to the permissions granted by the API
- Cannot access the file system directly (in browser environments)
- Must work within the iframe sandbox
- Performance depends on browser capabilities
- Some API methods require specific editor versions

## Conclusion

OnlyOffice plugins are versatile tools that empower both users and developers. They bridge the gap between standard office functionality and specialized needs, creating a customizable and extensible office suite. Understanding what plugins are and how they work is the foundation for creating powerful extensions that enhance productivity and workflow.

Ready to understand how plugins work throughout their lifecycle? Continue to the next section on Plugin Lifecycle.
