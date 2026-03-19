---
sidebar_position: -2
---

# 本地部署安装

ONLYOFFICE Docs（文档服务器）可以自托管，用于搭建类生产开发环境。最快的入门方式是使用 Docker。

## 使用 Docker 快速启动

```bash
docker run -i -t -d -p 80:80 --restart=always onlyoffice/documentserver
```

通过 `http://localhost` 访问编辑器。要测试插件，请使用容器中包含的示例集成页面：

```
http://localhost/example
```

## 添加插件

1. 从示例页面打开编辑器。
2. 转到**插件 → 插件管理器 → 通过 URL 添加插件**。
3. 输入插件 `config.json` 的 URL。

如需全组织范围内的部署，请在 ONLYOFFICE Docs 管理面板的**插件**设置中添加插件路径。

## 其他资源

- [ONLYOFFICE Docs Docker 安装](https://hub.docker.com/r/onlyoffice/documentserver)
- [适用于网页编辑器](../developing/for-web-editors.md) — 开发工作流
- [测试环境搭建](./test-environment-setup.md)
