---
sidebar_position: -3
---

# Cloud/SaaS 安装

ONLYOFFICE Cloud 提供托管环境，可在无需管理基础设施的情况下测试插件。在 [onlyoffice.com/registration.aspx](https://www.onlyoffice.com/registration.aspx) 注册以获取门户。

## 添加插件

1. 在您的 ONLYOFFICE Cloud 门户中打开文档。
2. 转到**插件 → 插件管理器 → 通过 URL 添加插件**。
3. 输入插件 `config.json` 的 URL。

:::note
`config.json` 的 URL 必须可公开访问。对于本地开发，请使用 `ngrok` 等隧道工具暴露本地服务器。
:::

## 全门户部署

门户管理员可以为所有用户部署插件：

1. 打开**门户设置 → 集成 → 插件**。
2. 添加 `config.json` 的 URL。
3. 保存——插件将对所有门户用户可用。

## 其他资源

- [适用于网页编辑器](../developing/for-web-editors.md) — 开发工作流
- [测试环境搭建](./test-environment-setup.md)
