---
sidebar_position: -3
---

# Changing a theme

Starting from version 7.5, you can change a theme of the desktop editor tab. To do this, use the _portal:uitheme_ command of the [execCommand](./execcommand.md) method.

```ts
window.AscDesktopEditor.execCommand("portal:uitheme", editorTheme);
```

## editorTheme

Defines the editor theme settings. It can be set in two ways:

- **theme id** - the user sets the theme parameter by its id (**theme-light**, **theme-classic-light**, **theme-dark**, **theme-contrast-dark**);
- **default theme** - the default dark or light theme value will be set (**default-dark**, **default-light**). The default light theme is **theme-classic-light**.

The first option has higher priority.

Apart from the available editor themes, the user can also customize their own [color themes](https://helpcenter.onlyoffice.com/installation/docs-developer-change-theme.aspx) for the application interface.

Type: string

Example: "theme-dark"

## Example

```ts
window.AscDesktopEditor.execCommand("portal:uitheme", "theme-dark");
```

When the _portal:uitheme_ command is sent, the editor theme is changed to the one specified in the method parameters.

## Changing themes in plugins

Plugins can detect and react to the current editor theme to keep their appearance consistent with the interface.  
Use the following API to retrieve the current theme settings:

```ts
const theme = window.Asc.plugin.getTheme();
console.log(theme); // Returns object with theme colors and properties
```

## Applying CSS for different themes

You can apply different CSS styles depending on the active theme. To do this, check the theme id and switch CSS variables or classes accordingly.

```ts
const currentTheme = window.Asc.plugin.getTheme().type;

if (currentTheme === "dark" || currentTheme === "contrast-dark") {
  document.body.classList.add("dark-theme");
} else {
  document.body.classList.remove("dark-theme");
}
```

Then define your CSS rules:

```css
.dark-theme button {
  background-color: #333;
  color: #fff;
}
```

This ensures your plugin automatically adapts its styling when users switch between light and dark themes.
