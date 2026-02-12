# Configuration

## Overview

The `config.json` file is the heart of every ONLYOFFICE plugin. It defines the plugin's metadata, appearance, behavior, and integration with the editor.

## Basic Structure

Every plugin requires a `config.json` file with this minimum structure:

```json
{
  "name": "My Plugin",
  "guid": "asc.{YOUR-UNIQUE-GUID}",
  "version": "1.0.0",
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png"],
      "isViewer": false,
      "EditorsSupport": ["word"]
    }
  ]
}
```

## Required Fields

### name

The display name shown to users. Keep it short (2-4 words), use title case.

```json
{
  "name": "Text Highlighter"
}
```

### guid

A unique identifier that must be globally unique. Always start with `asc.` followed by a UUID in curly braces.

```json
{
  "guid": "asc.{07FD8DFA-DFE0-4089-AL24-0730933CC804}"
}
```

**Generate a GUID:** Use [guidgenerator.com](https://www.guidgenerator.com/) or command line `uuidgen`

### version

Semantic version: `MAJOR.MINOR.PATCH`

```json
{
  "version": "1.2.3"
}
```

### variations

Array of plugin configurations. Most plugins have one variation.

```json
{
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png"]
    }
  ]
}
```

## Essential Variation Properties

### url

Path to the main HTML file.

```json
{
  "url": "index.html"
}
```

### icons

Icon file paths for different display densities.

```json
{
  "icons": ["resources/icon.png", "resources/icon@2x.png"]
}
```

**Recommended sizes:** 48x48px (standard), 96x96px (retina)

### icons2

Theme-aware icons for light and dark modes.

```json
{
  "icons2": [
    {
      "style": "light",
      "100%": { "normal": "resources/light/icon.png" },
      "200%": { "normal": "resources/light/icon@2x.png" }
    },
    {
      "style": "dark",
      "100%": { "normal": "resources/dark/icon.png" },
      "200%": { "normal": "resources/dark/icon@2x.png" }
    }
  ]
}
```

### EditorsSupport

Specifies which editor types support your plugin.

```json
{
  "EditorsSupport": ["word", "cell", "slide"]
}
```

**Options:** `"word"`, `"cell"`, `"slide"`, `"pdf"`

### isViewer

Whether plugin works in viewer mode.

```json
{
  "isViewer": false
}
```

`false` = edit mode only, `true` = both edit and viewer modes

### isVisual

Whether the plugin has a UI.

```json
{
  "isVisual": true
}
```

`true` = has UI, `false` = background plugin

### isModal

Whether plugin blocks document interaction.

```json
{
  "isModal": true
}
```

`true` = modal dialog, `false` = non-blocking

### isInsideMode

Whether plugin opens as a side panel.

```json
{
  "isInsideMode": true
}
```

`true` = side panel, `false` = separate window

### type

Display type for panel mode.

```json
{
  "type": "panel"
}
```

**Options:** `"panel"` or `"window"`

### size

Default dimensions `[width, height]` in pixels.

```json
{
  "size": [350, 500]
}
```

**Recommendations:** Modal: 400-600w × 300-500h, Panel: 300-400w × 400-600h

### buttons

Footer buttons for modal plugins.

```json
{
  "buttons": [
    { "text": "OK", "primary": true },
    { "text": "Cancel", "primary": false }
  ]
}
```

**Button IDs:** First = 0, Second = 1, Cancel = -1

## Localization

### nameLocale

Translated plugin names.

```json
{
  "name": "Text Highlighter",
  "nameLocale": {
    "ru": "Выделение текста",
    "de": "Texthervorhebung",
    "fr": "Surligneur de texte",
    "es": "Resaltador de texto"
  }
}
```

### descriptionLocale

Translated descriptions.

```json
{
  "description": "Highlight text with colors",
  "descriptionLocale": {
    "ru": "Выделение текста цветом",
    "de": "Texte hervorheben"
  }
}
```

**Supported languages:** `en`, `ru`, `de`, `fr`, `es`, `pt-BR`, `it`, `ja`, `zh`, `cs`, `uk`

## Advanced Configuration

### initDataType

Data type received on initialization.

```json
{
  "initDataType": "text"
}
```

**Options:** `"none"`, `"text"`, `"html"`, `"ole"`

### initOnSelectionChanged

Auto-open plugin when user selects text.

```json
{
  "initOnSelectionChanged": true
}
```

### baseUrl

Base URL for remotely hosted plugins.

```json
{
  "baseUrl": "https://example.com/plugins/my-plugin/"
}
```

## Store Configuration

Configuration for marketplace listing.

```json
{
  "store": {
    "background": {
      "light": "#4A90E2",
      "dark": "#2E5C8A"
    },
    "screenshots": [
      "resources/store/screenshots/screen_1.png",
      "resources/store/screenshots/screen_2.png"
    ],
    "categories": ["work", "specAbilities"],
    "icons": {
      "light": "resources/store/icons",
      "dark": "resources/store/icons"
    }
  }
}
```

**Categories:** `"work"`, `"specAbilities"`, `"entertainment"`, `"devTools"`, `"integration"`

## Complete Example

```json
{
  "name": "Word Counter",
  "nameLocale": {
    "ru": "Счетчик слов",
    "de": "Wortzähler"
  },
  "guid": "asc.{12345678-1234-1234-1234-123456789ABC}",
  "version": "1.0.0",

  "variations": [
    {
      "description": "Count words, characters, and paragraphs",
      "descriptionLocale": {
        "ru": "Подсчет слов, символов и абзацев"
      },

      "url": "index.html",

      "icons": ["resources/icon.png", "resources/icon@2x.png"],

      "icons2": [
        {
          "style": "light",
          "100%": { "normal": "resources/light/icon.png" },
          "200%": { "normal": "resources/light/icon@2x.png" }
        },
        {
          "style": "dark",
          "100%": { "normal": "resources/dark/icon.png" },
          "200%": { "normal": "resources/dark/icon@2x.png" }
        }
      ],

      "EditorsSupport": ["word"],
      "isViewer": true,
      "isVisual": true,
      "isModal": true,
      "isInsideMode": false,

      "size": [400, 300],

      "buttons": [{ "text": "OK", "primary": true }],

      "initDataType": "text",
      "initOnSelectionChanged": false,

      "store": {
        "background": {
          "light": "#36A2EB",
          "dark": "#1E5A8E"
        },
        "screenshots": ["resources/store/screen1.png"],
        "categories": ["work"],
        "icons": {
          "light": "resources/store/icons",
          "dark": "resources/store/icons"
        }
      }
    }
  ]
}
```

## Common Mistakes

### Missing Required Fields

**Error name:** Missing guid field

:::warning[Wrong]

```json
{
  "name": "My Plugin",
  "version": "1.0.0"
}
```

:::

:::tip[Correct]

```json
{
  "name": "My Plugin",
  "guid": "asc.{UNIQUE-GUID}",
  "version": "1.0.0",
  "variations": [...]
}
```

:::

Error output: Plugin will not appear in the Plugins menu. No error message displayed to user.

### Invalid GUID Format

**Error name:** Missing "asc." prefix in GUID

:::warning[Wrong]

```json
{
  "guid": "{12345678-1234-1234-1234-123456789ABC}"
}
```

:::

:::tip[Correct]

```json
{
  "guid": "asc.{12345678-1234-1234-1234-123456789ABC}"
}
```

:::

Error output: Plugin may fail to load or conflict with other plugins. Check browser console for GUID-related errors.

### Wrong Button Configuration

**Error name:** Buttons defined for non-modal plugins

:::warning[Wrong]

```json
{
  "isModal": false,
  "isInsideMode": true,
  "buttons": [{ "text": "OK" }]
}
```

:::

:::tip[Correct]

```json
{
  "isModal": false,
  "isInsideMode": true,
  "buttons": []
}
```

:::

Error output: Buttons are ignored for panel plugins. No visible error, but unnecessary configuration.

### Missing Variations Array

**Error name:** Empty or missing variations

:::warning[Wrong]

```json
{
  "name": "My Plugin",
  "guid": "asc.{UNIQUE-GUID}",
  "version": "1.0.0"
}
```

:::

:::tip[Correct]

```json
{
  "name": "My Plugin",
  "guid": "asc.{UNIQUE-GUID}",
  "version": "1.0.0",
  "variations": [
    {
      "url": "index.html",
      "icons": ["icon.png"]
    }
  ]
}
```

:::

Error output: Plugin will not load. Configuration is considered invalid.

### Incorrect File Paths

**Error name:** Wrong icon path

:::warning[Wrong]

```json
{
  "icons": ["icon.png"]
}
```

:::

:::tip[Correct]

```json
{
  "icons": ["resources/icons/icon.png"]
}
```

:::

Error output: Plugin icon will not display. Default placeholder icon shown instead.

## Debugging Configuration

If your plugin doesn't appear:

1. **Validate JSON syntax** - Use [jsonlint.com](https://jsonlint.com/)
2. **Check GUID uniqueness** - Ensure no conflicts
3. **Verify file paths** - All paths must be correct
4. **Test EditorsSupport** - Test in a supported editor
5. **Review console** - Check browser DevTools for errors

## Best Practices

1. **Use unique GUIDs** - Never copy from examples
2. **Follow semantic versioning** - Track changes properly
3. **Support multiple editors** - Include all compatible editors
4. **Provide theme icons** - Use `icons2` for better integration
5. **Keep descriptions concise** - One clear sentence
6. **Test different sizes** - Verify UI works at specified dimensions
7. **Validate JSON** - Always validate before testing

## Conclusion

Proper configuration is fundamental to plugin development. Understanding these options and following best practices ensures your plugin integrates seamlessly with ONLYOFFICE editors.
