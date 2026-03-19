---
sidebar_position: -1
---

import DocCardList from '@theme/DocCardList';
import {useCurrentSidebarCategory} from '@docusaurus/theme-common';

# 自定义 AI 函数示例

以下示例将展示如何创建自定义 AI 函数，并使用这些函数扩展[内联 AI 代理](/docs/plugin-and-macros/ai/ai-agent/#usage)的功能，从而使其适应特定的使用场景。

## 文本文档编辑器

<DocCardList items={[...[...useCurrentSidebarCategory().items[0].items]]} />

## 电子表格编辑器

<DocCardList items={[...[...useCurrentSidebarCategory().items[1].items]]} />

## 演示文稿编辑器

<DocCardList items={[...[...useCurrentSidebarCategory().items[2].items]]} />

## 支持

如果您想就自定义 AI 函数请求功能或报告错误，请使用 [GitHub](https://github.com/ONLYOFFICE/onlyoffice.github.io/issues) 上的 issues 区。
