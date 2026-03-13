---
sidebar_position: -3
---

# Cloud/SaaS installation

ONLYOFFICE Cloud provides a hosted environment for testing plugins without managing infrastructure. Register at [onlyoffice.com/registration.aspx](https://www.onlyoffice.com/registration.aspx) to get a portal.

## Adding a plugin

1. Open a document in your ONLYOFFICE Cloud portal.
2. Go to **Plugins → Plugin Manager → Add plugin from URL**.
3. Enter the URL to your plugin's `config.json`.

:::note
The `config.json` URL must be publicly accessible. For local development, use a tunneling tool such as `ngrok` to expose your local server.
:::

## Portal-wide deployment

Portal administrators can deploy plugins for all users:

1. Open **Portal Settings → Integration → Plugins**.
2. Add the `config.json` URL.
3. Save — the plugin becomes available to all portal users.

## Additional resources

- [For web editors](../developing/for-web-editors.md) — Development workflow
- [Test environment setup](./test-environment-setup.md)
