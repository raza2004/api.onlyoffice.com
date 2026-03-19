---
sidebar_position: -4
---

# 测试环境搭建

完整的插件开发环境包括代码编辑器、本地 HTTP 服务器以及至少一个用于测试的 ONLYOFFICE 安装实例。

## 推荐工具

| 工具 | 用途 |
|------|---------|
| VS Code | 插件代码编辑 |
| Git | 版本控制 |
| Node.js | 本地 HTTP 服务器、构建工具 |
| ONLYOFFICE 桌面编辑器 | 无需服务器的本地测试 |
| Docker | 在本地运行 ONLYOFFICE Docs |

## 启动本地 HTTP 服务器

如需测试网页编辑器，请通过 HTTP 提供插件文件夹服务：

```bash
npx http-server ./my-plugin --port 8080 --cors
```

`--cors` 标志允许网页编辑器从不同来源加载插件。

## 测试清单

在发布插件前，请在每个目标环境中进行测试：

- [ ] ONLYOFFICE 桌面编辑器（Windows、macOS 或 Linux）
- [ ] ONLYOFFICE 网页编辑器（Chrome 或基于 Chromium 的浏览器）
- [ ] 本地部署的 ONLYOFFICE Docs（如针对企业用户）
- [ ] ONLYOFFICE Cloud（如针对云端用户）

## 其他资源

- [桌面编辑器安装](./desktop-editors-installation.md)
- [本地部署安装](./docs-on-premises-installation.md)
- [Cloud/SaaS 安装](./cloud-saas-installation.md)
- [浏览器开发者工具指南](../debugging/browser-devtools-guide.md)
