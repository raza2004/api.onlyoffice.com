---
sidebar_position: 5
---

# 自定义按钮

要处理在插件变体中（而非模态窗口/面板中）指定的[按钮](../structure/configuration/configuration.md#variationsbuttons)，请使用 **button** 方法。当任意插件按钮被点击时，将调用此函数。

## 参数

| 名称 | 类型 | 描述 |
| ---- | ---- | ----------- |
| id | `number` | 定义 *config.json* 文件的 [buttons](../structure/configuration/configuration.md#variationsbuttons) 数组中按钮的索引。如果 `id == -1`，则插件认为 **关闭** 窗口的叉形按钮已被点击，或其操作已被某种方式中断。 |
| windowId | `number` | 定义模态窗口中按钮的索引。 |

## 示例

``` ts
Asc.plugin.button = (id, windowId) => {
  if (!windowId) {
    return;
  }

  if (windowId === newWindow.id) {
    console.log("Plugin button");
  }
};
```
