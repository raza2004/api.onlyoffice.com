---
sidebar_position: -2
---

# Docs (on-premises) installation

ONLYOFFICE Docs (Document Server) can be self-hosted for a production-like development environment. The quickest way to get started is with Docker.

## Quick start with Docker

```bash
docker run -i -t -d -p 80:80 --restart=always onlyoffice/documentserver
```

Access the editor at `http://localhost`. To test your plugin, use a sample integration page included in the container:

```
http://localhost/example
```

## Adding a plugin

1. Open an editor from the example page.
2. Go to **Plugins → Plugin Manager → Add plugin from URL**.
3. Enter the URL to your plugin's `config.json`.

For organization-wide deployment, add the plugin path in the ONLYOFFICE Docs admin panel under **Plugins** settings.

## Additional resources

- [ONLYOFFICE Docs Docker installation](https://hub.docker.com/r/onlyoffice/documentserver)
- [For web editors](../developing/for-web-editors.md) — Development workflow
- [Test environment setup](./test-environment-setup.md)
