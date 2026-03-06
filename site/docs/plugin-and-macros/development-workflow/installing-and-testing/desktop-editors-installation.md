# Desktop Editors installation

## Overview

Installing ONLYOFFICE Desktop Editors is the first step in setting up your plugin development environment. Desktop Editors provides a local testing environment where you can quickly iterate on your plugins without needing a server setup.

## Why use Desktop Editors for development?

- **Instant testing** - Test plugins immediately without server deployment
- **Offline development** - Work without internet connection
- **Easy debugging** - Access browser DevTools for debugging
- **Quick iteration** - Make changes and test instantly
- **Cross-platform** - Available for Windows, macOS, and Linux

## System requirements

### Minimum requirements

- **Operating System**: Windows 7+, macOS 10.10+, or Linux (Ubuntu 16.04+, Debian, Fedora)
- **RAM**: 2 GB minimum
- **Disk Space**: 1 GB for installation
- **Processor**: 1 GHz or faster

### Recommended for development

- **RAM**: 4 GB or more
- **Disk Space**: 5 GB (includes space for plugins and cache)
- **Processor**: Multi-core processor (Intel i5 or equivalent)
- **Display**: 1920x1080 resolution or higher

## Installation steps

### Windows installation

#### Download the installer

1. Visit [https://www.onlyoffice.com/download-desktop.aspx](https://www.onlyoffice.com/download-desktop.aspx)
2. Click "Download" for Windows
3. Choose between:
   - **EXE installer** - Recommended for most users
   - **MSI installer** - For enterprise deployment

#### Install ONLYOFFICE Desktop Editors

1. Run the downloaded installer file
2. Click "Next" in the setup wizard
3. Accept the license agreement
4. Choose installation location (default is recommended)
5. Select additional options:
   - Create desktop shortcut
   - Associate file types with ONLYOFFICE
6. Click "Install"
7. Wait for installation to complete
8. Click "Finish" to launch ONLYOFFICE

#### Verify installation

```cmd
# Check installation path
cd "C:\Program Files\ONLYOFFICE\DesktopEditors"

# Verify executable exists
dir DesktopEditors.exe
```

### macOS installation

#### Download the installer

1. Visit [https://www.onlyoffice.com/download-desktop.aspx](https://www.onlyoffice.com/download-desktop.aspx)
2. Click "Download" for macOS
3. Download the `.dmg` file

#### Install ONLYOFFICE Desktop Editors

1. Open the downloaded `.dmg` file
2. Drag the ONLYOFFICE icon to the Applications folder
3. Wait for the copy to complete
4. Eject the DMG (right-click and select "Eject")
5. Open Applications folder
6. Double-click ONLYOFFICE to launch

#### First launch (macOS security)

If you see a security warning:

1. Open System Preferences
2. Go to Security & Privacy
3. Click "Open Anyway" next to the ONLYOFFICE message
4. Confirm by clicking "Open"

#### Verify installation

```bash
# Check installation
ls -la /Applications/ONLYOFFICE.app

# Verify plugins directory exists
ls -la /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins
```

### Linux installation

#### Ubuntu/Debian

```bash
# Update package list
sudo apt-get update

# Install required dependencies
sudo apt-get install apt-transport-https ca-certificates

# Add ONLYOFFICE repository GPG key
wget -qO - https://download.onlyoffice.com/GPG-KEY-ONLYOFFICE | sudo apt-key add -

# Add ONLYOFFICE repository
echo "deb https://download.onlyoffice.com/repo/debian squeeze main" | sudo tee /etc/apt/sources.list.d/onlyoffice.list

# Update package list
sudo apt-get update

# Install ONLYOFFICE Desktop Editors
sudo apt-get install onlyoffice-desktopeditors

# Verify installation
which onlyoffice-desktopeditors
```

#### Fedora/CentOS/RHEL

```bash
# Add ONLYOFFICE repository
sudo yum install https://download.onlyoffice.com/repo/centos/main/noarch/onlyoffice-repo.noarch.rpm

# Install ONLYOFFICE Desktop Editors
sudo yum install onlyoffice-desktopeditors

# Verify installation
which onlyoffice-desktopeditors
```

#### AppImage (Universal Linux)

```bash
# Download AppImage
wget https://download.onlyoffice.com/install/desktop/editors/linux/onlyoffice-desktopeditors.AppImage

# Make executable
chmod +x onlyoffice-desktopeditors.AppImage

# Run ONLYOFFICE
./onlyoffice-desktopeditors.AppImage
```

## Locating the plugins directory

After installation, you need to know where to place your plugins:

### Windows

**System installation:**
```
C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

**User installation:**
```
C:\Users\[USERNAME]\AppData\Local\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\
```

### macOS

```
/Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/
```

### Linux

**System installation:**
```
/opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```

**User installation:**
```
~/.local/share/ONLYOFFICE/DesktopEditors/editors/sdkjs-plugins/
```

## Setting up for development

### Create development workspace

Create a separate folder for plugin development:

```bash
# Create workspace
mkdir ~/onlyoffice-plugins
cd ~/onlyoffice-plugins

# Create first plugin folder
mkdir my-first-plugin
cd my-first-plugin
```

### Create symbolic link (recommended)

Link your development folder to the plugins directory:

**Windows (Run as Administrator):**
```cmd
mklink /D "C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\my-first-plugin" "C:\dev\my-first-plugin"
```

**macOS/Linux:**
```bash
ln -s ~/onlyoffice-plugins/my-first-plugin /Applications/ONLYOFFICE.app/Contents/Resources/editors/sdkjs-plugins/my-first-plugin
```

### Enable developer mode

To access browser DevTools for debugging:

**Windows/Linux:**
- Press `Ctrl+Shift+Alt+F12` to enable developer mode
- Right-click on a plugin and select "Inspect Element"

**macOS:**
- Press `Cmd+Option+Shift+F12` to enable developer mode
- Right-click on a plugin and select "Inspect Element"

## Testing the installation

### Create a test plugin

Create these files in your plugin folder:

**config.json:**
```json
{
  "name": "Installation Test",
  "guid": "asc.{test-installation-001}",
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
    <meta charset="UTF-8">
    <title>Installation Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
            text-align: center;
        }
        .success {
            color: #4caf50;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h2>Installation Test</h2>
    <p class="success">✓ ONLYOFFICE Desktop Editors is correctly installed!</p>
    <p>Your development environment is ready.</p>
    <button onclick="testAPI()">Test API</button>
    
    <script>
        function testAPI() {
            window.Asc.plugin.executeMethod("PasteText", ["Installation successful!"]);
        }
        
        window.Asc.plugin.init = function() {
            console.log("Test plugin initialized");
        };
    </script>
</body>
</html>
```

**icon.png:**
- Create or download a 48x48 pixel PNG image

### Verify plugin appears

1. Restart ONLYOFFICE Desktop Editors
2. Open a new document
3. Click on "Plugins" tab
4. Look for "Installation Test" plugin
5. Click it to open
6. Click "Test API" button
7. Verify text is inserted into the document

## Common installation issues

### Plugin folder not found

**Error name:** Cannot locate plugins directory

:::warning[Wrong]
```bash
# Wrong path - using incorrect directory
cd /opt/onlyoffice/plugins/
```
:::

:::tip[Correct]
```bash
# Correct path - exact location
cd /opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/
```
:::

Error output: Directory not found - plugins won't load if placed in wrong location.

### Permission denied errors

**Error name:** Insufficient permissions to access plugins folder

:::warning[Wrong]
```bash
# Attempting to create plugin without permissions
mkdir /opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/my-plugin
```
:::

:::tip[Correct]
```bash
# Using sudo for system directories
sudo mkdir /opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/my-plugin
sudo chmod -R 755 /opt/onlyoffice/desktopeditors/editors/sdkjs-plugins/my-plugin
```
:::

Error output: Permission denied - Cannot create directory.

### Plugin doesn't appear after installation

**Error name:** Plugin not visible in Plugins menu

**Common causes:**
- Plugin folder not in correct location
- ONLYOFFICE not restarted after adding plugin
- Invalid `config.json` file
- Missing required files

**Solutions:**

:::tip[Checklist]
```bash
# 1. Verify plugin location
ls -la /path/to/sdkjs-plugins/your-plugin

# 2. Check config.json is valid
cat /path/to/your-plugin/config.json | python -m json.tool

# 3. Verify required files exist
ls -la /path/to/your-plugin/
# Should see: config.json, index.html, icon.png

# 4. Check file permissions
chmod -R 755 /path/to/your-plugin/

# 5. Restart ONLYOFFICE completely
# Fully quit (not just close windows) and relaunch
```
:::

Error output: Plugin does not appear in menu after installation.

### Symbolic link not working

**Error name:** Symbolic link creation failed

:::warning[Wrong]
```cmd
# Windows - Not running as Administrator
mklink /D "C:\Program Files\ONLYOFFICE\..." "C:\dev\my-plugin"
```
:::

:::tip[Correct]
```cmd
# Windows - Run Command Prompt as Administrator first
# Then create symbolic link
mklink /D "C:\Program Files\ONLYOFFICE\DesktopEditors\editors\sdkjs-plugins\my-plugin" "C:\dev\my-plugin"
```
:::

Error output: "You do not have sufficient privilege to perform this operation."

## Updating ONLYOFFICE Desktop Editors

To update to the latest version:

**Windows:**
- Download the latest installer
- Run it (existing installation will be detected)
- Choose "Update" option

**macOS:**
- Download the latest `.dmg`
- Drag to Applications (replace existing)

**Linux:**
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get upgrade onlyoffice-desktopeditors

# Fedora/CentOS
sudo yum update onlyoffice-desktopeditors
```

**Note:** Your plugins will be preserved during updates.

## Uninstalling (if needed)

**Windows:**
- Control Panel → Programs → Uninstall a program
- Select ONLYOFFICE Desktop Editors
- Click Uninstall

**macOS:**
- Drag ONLYOFFICE from Applications to Trash
- Empty Trash

**Linux:**
```bash
# Ubuntu/Debian
sudo apt-get remove onlyoffice-desktopeditors

# Fedora/CentOS
sudo yum remove onlyoffice-desktopeditors
```

## Next steps

Now that ONLYOFFICE Desktop Editors is installed:

1. Set up your [test environment](./test-environment-setup.md)
2. Create your first plugin
3. Learn about the [plugin development workflow](../overview.md)
4. Explore the [plugin API documentation](/docs/plugin-and-macros/interacting-with-editors/overview.md)

## Additional resources

- **Official documentation**: [https://helpcenter.onlyoffice.com](https://helpcenter.onlyoffice.com)
- **Download page**: [https://www.onlyoffice.com/download-desktop.aspx](https://www.onlyoffice.com/download-desktop.aspx)
- **Community forum**: [https://forum.onlyoffice.com](https://forum.onlyoffice.com)
- **GitHub**: [https://github.com/ONLYOFFICE](https://github.com/ONLYOFFICE)

## Conclusion

ONLYOFFICE Desktop Editors provides an ideal environment for plugin development with quick setup, easy testing, and powerful debugging capabilities. With the installation complete, you're ready to start building plugins!
