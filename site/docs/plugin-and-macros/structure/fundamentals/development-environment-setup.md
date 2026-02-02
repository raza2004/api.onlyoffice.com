---
sidebar_position: 4
---

# Development environment setup

## Overview

Setting up a proper development environment is crucial for efficient OnlyOffice plugin development. This guide will walk you through configuring your workspace, installing necessary tools, and establishing a productive development workflow.

## System Requirements

### Minimum Requirements

- **Operating System**: Windows 7+, macOS 10.10+, or Linux (Ubuntu 16.04+, Debian, Fedora)
- **RAM**: 2 GB minimum, 4 GB recommended
- **Disk Space**: 1 GB for OnlyOffice installation plus space for development tools
- **Internet Connection**: Required for downloading tools and testing external integrations

### Recommended Specifications

- **RAM**: 8 GB or more
- **Processor**: Multi-core processor (Intel i5 or equivalent)
- **Disk Space**: SSD with at least 5 GB free space
- **Display**: 1920x1080 resolution or higher

## Step 1: Install OnlyOffice

### Option A: Desktop Editors (Recommended for Development)

OnlyOffice Desktop Editors is the easiest option for plugin development as it provides a local environment with immediate feedback.

#### Windows

1. Download the installer from [https://www.onlyoffice.com/download-desktop.aspx](https://www.onlyoffice.com/download-desktop.aspx)
2. Run the `.exe` installer
3. Follow the installation wizard
4. Launch OnlyOffice Desktop Editors

#### macOS

1. Download the `.dmg` file from the OnlyOffice website
2. Open the DMG file
3. Drag OnlyOffice to your Applications folder
4. Launch from Applications

#### Linux (Ubuntu/Debian)

```bash
# Add the repository
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates

# Add the GPG key
wget -qO - https://download.onlyoffice.com/GPG-KEY-ONLYOFFICE | sudo apt-key add -

# Add the repository
echo "deb https://download.onlyoffice.com/repo/debian squeeze main" | sudo tee /etc/apt/sources.list.d/onlyoffice.list

# Install OnlyOffice Desktop Editors
sudo apt-get update
sudo apt-get install onlyoffice-desktopeditors
```

#### Linux (Fedora/CentOS)

```bash
# Add the repository
sudo yum install https://download.onlyoffice.com/repo/centos/main/noarch/onlyoffice-repo.noarch.rpm

# Install OnlyOffice Desktop Editors
sudo yum install onlyoffice-desktopeditors
```

### Option B: Document Server (For Production Testing)

For testing plugins in a server environment:

#### Docker Installation (Recommended)

```bash
# Pull the OnlyOffice Document Server image
docker pull onlyoffice/documentserver

# Run the container
docker run -i -t -d -p 80:80 \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
    -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
    onlyoffice/documentserver
```

#### Manual Installation

Follow the detailed guide at [https://helpcenter.onlyoffice.com/installation/docs-developer-install.aspx](https://helpcenter.onlyoffice.com/installation/docs-developer-install.aspx)

## Step 2: Install Development Tools

### Text Editor or IDE

Choose one of the following development environments:

#### Visual Studio Code (Recommended)

**Why VS Code?**

- Excellent JavaScript support
- Integrated terminal
- Git integration
- Extension marketplace
- Free and open-source

**Installation:**

1. Download from [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. Install for your operating system
3. Launch VS Code

**Recommended Extensions:**

```bash
# Open VS Code and install these extensions:
# - ESLint (for code quality)
# - Prettier (for code formatting)
# - Live Server (for HTML preview)
# - JavaScript (ES6) code snippets
# - Path Intellisense
# - GitLens
```

Install extensions via command palette (`Ctrl+Shift+P` / `Cmd+Shift+P`):

```
ext install dbaeumer.vscode-eslint
ext install esbenp.prettier-vscode
ext install ritwickdey.LiveServer
ext install xabikos.JavaScriptSnippets
ext install christian-kohler.path-intellisense
ext install eamodio.gitlens
```

#### Other Options

**Sublime Text:**

- Fast and lightweight
- Download from [https://www.sublimetext.com/](https://www.sublimetext.com/)

**WebStorm:**

- Professional IDE with advanced features
- Free for students and open-source projects
- Download from [https://www.jetbrains.com/webstorm/](https://www.jetbrains.com/webstorm/)

**Atom:**

- Hackable text editor
- Download from [https://atom.io/](https://atom.io/)

### Web Browser with Developer Tools

Install a modern browser with robust developer tools:

**Google Chrome (Recommended)**

- Download from [https://www.google.com/chrome/](https://www.google.com/chrome/)
- Excellent DevTools for debugging
- React Developer Tools extension available

**Firefox Developer Edition**

- Download from [https://www.mozilla.org/firefox/developer/](https://www.mozilla.org/firefox/developer/)
- Great for testing and debugging

**Microsoft Edge**

- Built-in on Windows 10/11
- Chromium-based with good DevTools

### Node.js and npm

While not strictly required for basic plugins, Node.js is essential for advanced development:

**Installation:**

1. Download from [https://nodejs.org/](https://nodejs.org/)
2. Choose the LTS (Long Term Support) version
3. Run the installer for your OS

**Verify Installation:**

```bash
node --version
# Should output: v18.x.x or higher

npm --version
# Should output: 9.x.x or higher
```

**Global npm Packages (Optional but Useful):**

```bash
# HTTP server for local testing
npm install -g http-server

# Code linting
npm install -g eslint

# Code formatting
npm install -g prettier

# Build tools
npm install -g webpack webpack-cli
```

### Git (Version Control)

**Installation:**

**Windows:**

- Download from [https://git-scm.com/download/win](https://git-scm.com/download/win)
- Run the installer
- Use default settings

**macOS:**

```bash
# Install via Homebrew
brew install git

# Or download from https://git-scm.com/download/mac
```

**Linux:**

```bash
# Ubuntu/Debian
sudo apt-get install git

# Fedora
sudo yum install git
```

**Configuration:**

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## Step 3: Locate and Access Plugin Directories

### Find Your Plugins Directory

The location varies by operating system and installation type:

#### OnlyOffice Desktop Editors

**Windows:**

```
C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

Or for user-specific installation:

```
C:\Users\[USERNAME]\AppData\Local\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

**macOS:**

```
/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
```

**Linux:**

```
/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```

Or for user installation:

```
~/.local/share/ONLYOFFICE/DesktopEditors/editors/sdkjs-plugins/
```

#### OnlyOffice Document Server

**Docker:**

```
# Inside container at:
/var/www/onlyoffice/documentserver/sdkjs-plugins/

# Or mount a volume:
docker run -v /your/plugins:/var/www/onlyoffice/documentserver/sdkjs-plugins onlyoffice/documentserver
```

**Manual Installation:**

```
/var/www/onlyoffice/documentserver/sdkjs-plugins/
```

### Set Up a Development Workspace

Create a dedicated development folder outside the OnlyOffice directory:

```bash
# Create workspace
mkdir ~/onlyoffice-plugins-dev
cd ~/onlyoffice-plugins-dev

# Create a sample plugin structure
mkdir my-first-plugin
cd my-first-plugin
```

## Step 4: Configure Your Development Workflow

### Option A: Symbolic Links (Recommended)

Create symbolic links from your development folder to the OnlyOffice plugins directory:

**Windows (Run as Administrator):**

```cmd
mklink /D "C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\my-plugin" "C:\dev\my-plugin"
```

**macOS/Linux:**

```bash
ln -s ~/onlyoffice-plugins-dev/my-plugin /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/my-plugin
```

**Benefits:**

- Edit files in your workspace
- Changes immediately available in OnlyOffice
- Easy version control

### Option B: Manual Copy Workflow

Create a script to copy files to the plugins directory:

**deploy.sh (macOS/Linux):**

```bash
#!/bin/bash

PLUGIN_NAME="my-plugin"
DEV_DIR="$HOME/onlyoffice-plugins-dev/$PLUGIN_NAME"
ONLYOFFICE_PLUGINS="/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins"

echo "Deploying $PLUGIN_NAME..."

# Remove old version
rm -rf "$ONLYOFFICE_PLUGINS/$PLUGIN_NAME"

# Copy new version
cp -r "$DEV_DIR" "$ONLYOFFICE_PLUGINS/"

echo "Deployment complete!"
echo "Please restart OnlyOffice to see changes."
```

**deploy.bat (Windows):**

```batch
@echo off
set PLUGIN_NAME=my-plugin
set DEV_DIR=C:\dev\%PLUGIN_NAME%
set ONLYOFFICE_PLUGINS=C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins

echo Deploying %PLUGIN_NAME%...

rd /s /q "%ONLYOFFICE_PLUGINS%\%PLUGIN_NAME%"
xcopy /E /I "%DEV_DIR%" "%ONLYOFFICE_PLUGINS%\%PLUGIN_NAME%"

echo Deployment complete!
echo Please restart OnlyOffice to see changes.
pause
```

Make executable:

```bash
chmod +x deploy.sh
```

### Option C: Watch and Auto-Deploy

Use Node.js to watch for changes and auto-deploy:

**package.json:**

```json
{
  "name": "onlyoffice-plugin-dev",
  "version": "1.0.0",
  "scripts": {
    "watch": "node watch.js"
  },
  "devDependencies": {
    "chokidar": "^3.5.3",
    "fs-extra": "^11.1.0"
  }
}
```

**watch.js:**

```javascript
const chokidar = require("chokidar");
const fs = require("fs-extra");
const path = require("path");

const SOURCE_DIR = path.join(__dirname, "my-plugin");
const TARGET_DIR =
  "/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/my-plugin";

console.log("Watching for changes...");

const watcher = chokidar.watch(SOURCE_DIR, {
  ignored: /(^|[\/\\])\../, // ignore dotfiles
  persistent: true,
});

watcher.on("change", (filePath) => {
  console.log(`File changed: ${filePath}`);
  deployPlugin();
});

function deployPlugin() {
  fs.copy(SOURCE_DIR, TARGET_DIR, (err) => {
    if (err) {
      console.error("Deployment failed:", err);
    } else {
      console.log("Plugin deployed! Restart OnlyOffice to see changes.");
    }
  });
}

// Initial deployment
deployPlugin();
```

Install and run:

```bash
npm install
npm run watch
```

## Step 5: Set Up Debugging Tools

### Browser DevTools for Plugin Debugging

#### Enable DevTools in OnlyOffice Desktop

**Windows/Linux:**

1. Open OnlyOffice Desktop Editors
2. Press `Ctrl+Shift+Alt+F12` to enable developer mode
3. Right-click on a plugin and select "Inspect Element"

**macOS:**

1. Open OnlyOffice Desktop Editors
2. Press `Cmd+Option+Shift+F12`
3. Right-click on a plugin and select "Inspect Element"

#### Debug Plugin Code

```javascript
// Add console logs
console.log("Plugin initialized");
console.log("Data received:", data);

// Add breakpoints in browser DevTools
debugger; // Execution will pause here

// Check errors
try {
  window.Asc.plugin.executeMethod("PasteText", ["Hello"]);
} catch (error) {
  console.error("Error:", error);
}
```

### Create a Debug Configuration

**.vscode/launch.json** (for VS Code):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Plugin in Chrome",
      "url": "file://${workspaceFolder}/index.html",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

## Step 6: Create a Project Template

### Plugin Boilerplate Structure

```bash
my-plugin/
├── .vscode/
│   ├── launch.json
│   └── settings.json
├── assets/
│   ├── icons/
│   │   ├── icon.png
│   │   ├── icon@2x.png
│   │   └── icon_dark.png
│   └── styles/
│       └── main.css
├── src/
│   ├── plugin.js
│   └── utils.js
├── config.json
├── index.html
├── README.md
└── .gitignore
```

**.gitignore:**

```
node_modules/
.DS_Store
Thumbs.db
*.log
.vscode/
```

**package.json** (if using npm):

```json
{
  "name": "my-onlyoffice-plugin",
  "version": "1.0.0",
  "description": "My OnlyOffice Plugin",
  "scripts": {
    "dev": "node watch.js",
    "deploy": "bash deploy.sh",
    "lint": "eslint src/**/*.js"
  },
  "author": "Your Name",
  "license": "MIT",
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^2.0.0"
  }
}
```

### Code Quality Tools

**.eslintrc.json:**

```json
{
  "env": {
    "browser": true,
    "es6": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", 2],
    "quotes": ["error", "double"],
    "semi": ["error", "always"]
  },
  "globals": {
    "Asc": "readonly"
  }
}
```

**.prettierrc:**

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 80,
  "tabWidth": 2
}
```

## Step 7: Test Your Setup

### Create a Test Plugin

Create a minimal plugin to verify your setup:

**config.json:**

```json
{
  "name": "Setup Test",
  "guid": "asc.{test-setup-plugin}",
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

**index.html:**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Setup Test</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        padding: 20px;
      }
      .success {
        color: green;
        font-weight: bold;
      }
    </style>
  </head>
  <body>
    <h2>Development Environment Test</h2>
    <p class="success">✓ Your development environment is working!</p>
    <button onclick="testPlugin()">Test Plugin API</button>
    <script>
      function testPlugin() {
        window.Asc.plugin.executeMethod("PasteText", ["Setup successful!"]);
      }

      window.Asc.plugin.init = function () {
        console.log("Test plugin initialized successfully!");
      };
    </script>
  </body>
</html>
```

### Verify the Setup

1. Deploy the test plugin to OnlyOffice plugins directory
2. Restart OnlyOffice Desktop Editors
3. Create a new document
4. Open the Plugins tab
5. Look for "Setup Test" plugin
6. Click it and verify it loads
7. Click "Test Plugin API" button
8. Verify text is inserted into the document

## Troubleshooting Common Setup Issues

### Plugin Doesn't Appear

**Check:**

- `config.json` is valid JSON (use jsonlint.com)
- Plugin folder is in the correct directory
- OnlyOffice was restarted after adding the plugin
- File permissions allow OnlyOffice to read the files

### Plugin Loads But Doesn't Work

**Check:**

- Browser console for JavaScript errors (enable DevTools)
- Network tab for failed resource loads
- `window.Asc.plugin` is available
- All file paths in config.json are correct

### Changes Not Showing Up

**Solutions:**

- Clear OnlyOffice cache
- Fully quit and restart OnlyOffice (not just close windows)
- Verify symbolic link is working (if used)
- Check file timestamps to ensure files were copied

### Permission Errors

**Windows:**

- Run command prompt as Administrator
- Check file permissions on plugins folder

**macOS/Linux:**

```bash
# Fix permissions
sudo chmod -R 755 /path/to/plugins/directory
sudo chown -R $USER /path/to/plugins/directory
```

## Next Steps

Now that your development environment is set up, you're ready to:

1. Create your first plugin (see "Your First Plugin" guide)
2. Explore the OnlyOffice API documentation
3. Study example plugins
4. Join the OnlyOffice developer community

## Additional Resources

- **OnlyOffice API Documentation**: https://api.onlyoffice.com/plugin/basic
- **GitHub Examples**: https://github.com/ONLYOFFICE/sdkjs-plugins
- **Developer Forum**: https://forum.onlyoffice.com/
- **Plugin Marketplace**: https://www.onlyoffice.com/app-directory

Your development environment is now ready for OnlyOffice plugin development!
