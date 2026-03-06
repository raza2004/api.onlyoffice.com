# Cloud/SaaS installation

## Overview

ONLYOFFICE Cloud (SaaS) provides a hosted solution where you can test plugins without managing infrastructure. This is ideal for quick testing, collaboration, and demonstrating plugins to stakeholders without setting up local or on-premises environments.

## Why use ONLYOFFICE Cloud for plugin development?

- **Zero setup** - No installation required, start immediately
- **Always updated** - Latest version automatically available
- **Collaboration** - Easy sharing with team members
- **Cross-device testing** - Test on any device with browser
- **Cost-effective** - Free tier available for testing
- **Production-like** - Test in real cloud environment

## Getting started

### Create ONLYOFFICE Cloud account

1. Visit [https://www.onlyoffice.com/registration.aspx](https://www.onlyoffice.com/registration.aspx)
2. Fill in registration form:
   - Email address
   - Password
   - Portal name (e.g., mycompany.onlyoffice.com)
3. Verify your email
4. Complete profile setup
5. Access your cloud portal

### Choose subscription plan

ONLYOFFICE Cloud offers different plans:

**Free Plan:**
- Up to 5 users
- 2 GB storage
- Basic features
- Perfect for plugin testing

**Business Plans:**
- More users
- Increased storage
- Advanced features
- Priority support

For development purposes, the free plan is usually sufficient.

## Accessing the editor

### Via web interface

1. Log in to your ONLYOFFICE Cloud portal
2. Click "Documents"
3. Create or upload a document
4. Document opens in the online editor

### Direct document URL

```
https://your-portal.onlyoffice.com/products/files/#preview/document-id
```

## Installing plugins in ONLYOFFICE Cloud

### Understanding plugin limitations

:::warning[Important]
ONLYOFFICE Cloud has restrictions on custom plugin installation:

- **Cannot upload custom plugins directly** to free/standard accounts
- **Enterprise plans** may support custom plugins
- **Plugin marketplace** plugins are available to all users
- **Local testing** recommended for custom plugin development
:::

### Using marketplace plugins

1. Open any document in the editor
2. Click "Plugins" tab in the top toolbar
3. Browse available plugins
4. Click "Install" on desired plugin
5. Plugin appears in your plugins list

### Testing custom plugins (workaround)

Since custom plugin upload is limited in Cloud, use these alternatives:

**Option 1: Use Desktop Editors with Cloud**
```javascript
// Connect Desktop Editors to Cloud
// Settings → Cloud Services → Connect to ONLYOFFICE Cloud
// Test plugins locally while accessing cloud documents
```

**Option 2: Request Enterprise trial**
- Contact ONLYOFFICE sales
- Request enterprise trial with custom plugin support
- Test your plugins in cloud environment

**Option 3: Hybrid approach**
- Develop locally with Desktop Editors
- Deploy to on-premises for full testing
- Use Cloud for final user testing

## Integrating ONLYOFFICE Cloud with your app

### Using ONLYOFFICE Cloud API

**Get API credentials:**
1. Open your portal settings
2. Navigate to "API" section
3. Generate API key
4. Note your portal URL

**Basic integration:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>ONLYOFFICE Cloud Integration</title>
    <script src="https://your-portal.onlyoffice.com/web-apps/apps/api/documents/api.js"></script>
</head>
<body>
    <div id="placeholder"></div>
    <script>
        new DocsAPI.DocEditor("placeholder", {
            "document": {
                "fileType": "docx",
                "key": "unique-key-123",
                "title": "My Document.docx",
                "url": "https://your-server.com/document.docx",
                "permissions": {
                    "edit": true,
                    "download": true
                }
            },
            "documentType": "word",
            "editorConfig": {
                "mode": "edit",
                "callbackUrl": "https://your-server.com/callback",
                "user": {
                    "id": "user-id",
                    "name": "John Doe"
                }
            },
            "height": "600px",
            "width": "100%"
        });
    </script>
</body>
</html>
```

### Testing plugins with embedded editor

Even though you can't upload custom plugins to Cloud, you can test plugin behavior:

1. Use marketplace plugins as examples
2. Study their behavior in Cloud
3. Develop similar functionality locally
4. Deploy to on-premises for full testing

## Collaborative testing

### Share documents for testing

1. Create test document in Cloud
2. Click "Share" button
3. Invite team members by email
4. Set permissions (view/edit)
5. Collaborate on testing

### Test plugin with multiple users

**Scenario: Testing collaborative features**

```javascript
// Plugin that tracks user interactions
window.Asc.plugin.init = function() {
    // Get current user info
    window.Asc.plugin.info = {
        userId: getCurrentUserId(),
        userName: getCurrentUserName()
    };
    
    // Test collaborative features
    window.Asc.plugin.attachEvent("onUserChanged", function(userData) {
        console.log("User changed:", userData);
    });
};
```

## Monitoring and analytics

### Access usage statistics

1. Go to portal settings
2. Open "Statistics" section
3. View:
   - Active users
   - Document edits
   - Storage usage
   - API calls

### Track plugin usage

For marketplace plugins:
```javascript
// Plugin can send analytics
window.Asc.plugin.init = function() {
    trackPluginUsage({
        pluginId: 'my-plugin',
        action: 'opened',
        timestamp: new Date()
    });
};
```

## Common scenarios

### Testing plugin in different browsers

**Desktop browsers:**
- Chrome/Edge (Chromium)
- Firefox
- Safari
- Opera

**Mobile browsers:**
- Chrome Mobile
- Safari Mobile
- Samsung Internet

**Test checklist:**

:::tip[Browser testing checklist]
```
□ Chrome - Latest version
□ Firefox - Latest version
□ Safari - Latest version
□ Edge - Latest version
□ Mobile Chrome - iOS and Android
□ Mobile Safari - iOS
□ Tablet browsers
□ Different screen resolutions
```
:::

### Testing plugin performance in cloud

```javascript
// Add performance monitoring to plugin
window.Asc.plugin.init = function() {
    const startTime = performance.now();
    
    // Plugin initialization code
    initializePlugin();
    
    const endTime = performance.now();
    const loadTime = endTime - startTime;
    
    console.log(`Plugin loaded in ${loadTime}ms`);
    
    // Send metrics if needed
    if (loadTime > 1000) {
        console.warn("Plugin loading slowly");
    }
};
```

## Limitations and considerations

### Cloud-specific limitations

**Cannot do in Cloud:**
- Upload custom plugins (free/standard plans)
- Access server-side configurations
- Modify editor core functionality
- Install third-party dependencies globally

**Can do in Cloud:**
- Use marketplace plugins
- Test plugin UI/UX concepts
- Collaborate with team
- Access documents anywhere
- Test browser compatibility

### Data and privacy

**Important considerations:**

:::warning[Data handling]
- Data stored in ONLYOFFICE Cloud servers
- Subject to ONLYOFFICE privacy policy
- Consider data residency requirements
- Use on-premises for sensitive data
- Review compliance requirements (GDPR, HIPAA, etc.)
:::

### Network requirements

**Required connectivity:**
- Stable internet connection
- Minimum 1 Mbps upload/download
- Low latency (&lt;100ms recommended)
- WebSocket support

**Test network conditions:**

```javascript
// Check connection quality
window.addEventListener('online', function() {
    console.log('Connection restored');
});

window.addEventListener('offline', function() {
    console.warn('Connection lost');
});

// Monitor connection speed
if (navigator.connection) {
    console.log('Connection type:', navigator.connection.effectiveType);
    console.log('Downlink speed:', navigator.connection.downlink);
}
```

## Troubleshooting

### Cannot access cloud portal

**Error name:** Portal not accessible

:::warning[Wrong]
```
# Trying to access with wrong URL
https://mycompany.office.com
```
:::

:::tip[Correct]
```
# Correct URL format
https://mycompany.onlyoffice.com

# Or personal portal
https://personal.onlyoffice.com
```
:::

Error output: "Site can't be reached" or 404 error.

### Plugin not appearing in Cloud

**Error name:** Custom plugin not visible

**Cause:** Cloud doesn't support custom plugin upload on free/standard plans.

:::tip[Solution]
```
1. Use ONLYOFFICE Desktop Editors for development
2. Test custom plugins locally
3. Use Cloud for:
   - Marketplace plugins testing
   - Collaboration testing
   - Browser compatibility
4. Consider enterprise plan for custom plugin support
5. Or use on-premises installation
```
:::

Error output: No option to upload custom plugins in settings.

### Editor not loading documents

**Error name:** Document fails to open

:::warning[Wrong]
```javascript
// Incorrect document URL format
{
  "document": {
    "url": "file:///C:/documents/test.docx"
  }
}
```
:::

:::tip[Correct]
```javascript
// Correct - use publicly accessible URL
{
  "document": {
    "url": "https://your-server.com/documents/test.docx"
  }
}
```
:::

Error output: "Error loading document" or "Cannot access file".

### Slow performance in Cloud

**Error name:** Editor sluggish or unresponsive

**Common causes:**
- Slow internet connection
- Large document size
- Browser issues
- Server region distance

:::tip[Solutions]
```
1. Check internet speed (minimum 1 Mbps)
2. Optimize document size:
   - Reduce image quality
   - Remove unused elements
   - Split large documents

3. Clear browser cache

4. Try different browser

5. Check server status:
   https://status.onlyoffice.com

6. Contact support if persistent
```
:::

Error output: Slow typing, delayed actions, timeout errors.

## Best practices for cloud testing

### Document management

```javascript
// Keep test documents organized
const testDocuments = {
    unit: [
        'basic-text-test.docx',
        'formatting-test.docx',
        'table-test.docx'
    ],
    integration: [
        'multi-user-test.docx',
        'large-document-test.docx'
    ],
    performance: [
        'stress-test.docx'
    ]
};
```

### Version control for test files

1. Keep test documents in Cloud folder
2. Name clearly: `test-plugin-feature-v1.docx`
3. Archive old versions
4. Document test results

### Collaborative testing workflow

```
1. Developer creates test document
2. Share with QA team
3. QA team tests plugin features
4. Document issues in comments
5. Developer fixes and updates
6. Repeat until approved
```

## Migration paths

### From Cloud to On-premises

When ready to deploy custom plugins:

1. Export documents from Cloud
2. Set up on-premises installation
3. Import documents
4. Install custom plugins
5. Test thoroughly
6. Train users

### From Desktop to Cloud

For sharing plugins with users:

1. Package plugin properly
2. Submit to ONLYOFFICE marketplace
3. Wait for approval
4. Cloud users can install from marketplace

## Next steps

- Learn about [test environment setup](./test-environment-setup.md)
- Explore [Desktop Editors installation](./desktop-editors-installation.md)
- Consider [on-premises installation](./docs-on-premises-installation.md) for custom plugins
- Join [ONLYOFFICE community](https://forum.onlyoffice.com)

## Additional resources

- **ONLYOFFICE Cloud**: [https://www.onlyoffice.com/cloud-office.aspx](https://www.onlyoffice.com/cloud-office.aspx)
- **API documentation**: [https://api.onlyoffice.com](https://api.onlyoffice.com)
- **Help center**: [https://helpcenter.onlyoffice.com](https://helpcenter.onlyoffice.com)
- **Status page**: [https://status.onlyoffice.com](https://status.onlyoffice.com)

## Conclusion

ONLYOFFICE Cloud provides a convenient testing environment for plugin concepts, browser compatibility, and collaboration features. While custom plugin installation is limited, it's valuable for early-stage testing and demonstrating plugin ideas before deploying to production environments.