---
sidebar_position: -4
---

# 使用选区

## 概述

处理文本和对象选区是 ONLYOFFICE 插件开发的基础环节。本指南介绍如何在不同编辑器类型中检测、操作和响应用户选区。

## 理解选区

选区代表用户在文档、电子表格或演示文稿中当前高亮显示的内容。插件可以：

- **读取选区内容** - 获取用户所选的文本、格式或对象
- **修改选区** - 以编程方式更改所选内容
- **响应变化** - 在用户更改选区时作出响应
- **替换选区** - 在所选内容的位置插入新内容

## 获取所选内容

### 获取所选文本

最常见的操作是检索用户所选的文本：

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text && text.trim() !== "") {
    console.log("Selected text:", text);
    // Process the selected text
  } else {
    console.log("No text selected");
  }
});
```

### 获取选区范围

要获取关于选区的更详细信息：

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    const text = range.GetText();
    const length = text.length;
    console.log(`Selected: ${length} characters`);
    return { text: text, length: length };
  }
  return null;
}, false);
```

### 获取带格式的内容

检索包含格式信息的选区：

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Get text properties
    const textPr = range.GetTextPr();

    return {
      text: range.GetText(),
      isBold: textPr.GetBold(),
      isItalic: textPr.GetItalic(),
      fontSize: textPr.GetFontSize(),
    };
  }
  return null;
}, false);
```

## 响应选区变化

### 使用 onSelectionChanged 事件

监听用户更改选区的时机：

```javascript
window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  if (selection && selection.text) {
    console.log("New selection:", selection.text);
    updatePluginUI(selection.text);
  } else {
    console.log("Selection cleared");
    clearPluginUI();
  }
});
```

### 选中文本时自动打开

配置插件在选中文本时自动打开：

**config.json:**

```json
{
  "initOnSelectionChanged": true,
  "initDataType": "text"
}
```

**plugin.js:**

```javascript
window.Asc.plugin.init = function (selectedText) {
  if (selectedText && selectedText.trim()) {
    // Plugin opened with selected text
    document.getElementById("input").value = selectedText;
    processText(selectedText);
  }
};
```

## 修改选区

### 替换所选文本

用新内容替换当前选区：

```javascript
function replaceSelection(newText) {
  window.Asc.plugin.executeMethod("PasteText", [newText], function () {
    console.log("Selection replaced successfully");
  });
}

// Example: Convert selection to uppercase
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    const upperText = text.toUpperCase();
    window.Asc.plugin.executeMethod("PasteText", [upperText]);
  }
});
```

### 替换为带格式的内容

用 HTML 或带格式的文本替换选区：

```javascript
function replaceWithFormatted(htmlContent) {
  window.Asc.plugin.executeMethod("PasteHtml", [htmlContent], function () {
    console.log("Formatted content inserted");
  });
}

// Example: Bold the selected text
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    const textPr = Api.CreateTextPr();
    textPr.SetBold(true);
    range.SetTextPr(textPr);
  }
}, false);
```

### 包裹选区

用附加内容包裹所选文本：

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    const wrapped = `**${text}**`; // Wrap in bold markdown
    window.Asc.plugin.executeMethod("PasteText", [wrapped]);
  }
});
```

## 在不同编辑器类型中使用选区

### 文档编辑器选区

在文字处理文档中，选区可以跨越：

- 文本字符
- 多个段落
- 表格和单元格
- 图片和形状

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Check if selection contains a table
    const parent = range.GetParent();
    if (parent && parent.GetClassType() === "docTable") {
      console.log("Selection is in a table");
    }
  }
}, false);
```

### 电子表格选区

在电子表格中，选区作用于单元格和范围：

```javascript
window.Asc.plugin.callCommand(function () {
  const sheet = Api.GetActiveSheet();
  const selection = sheet.GetSelection();

  // Get selected range address (e.g., "A1:C5")
  const address = selection.GetAddress();

  // Get values from selected cells
  const values = selection.GetValue();

  return { address: address, values: values };
}, false);
```

### 演示文稿选区

在演示文稿中，选区可以包括：

- 形状中的文本
- 整个形状
- 多个对象
- 幻灯片内容

```javascript
window.Asc.plugin.callCommand(function () {
  const presentation = Api.GetPresentation();
  const currentSlide = presentation.GetCurrentSlide();

  // Get all shapes on the current slide
  const shapes = currentSlide.GetAllShapes();

  // Work with selected shapes
  shapes.forEach(function (shape) {
    const content = shape.GetDocContent();
    // Process shape content
  });
}, false);
```

## 高级选区技术

### 多选区处理

处理多个不连续的选区：

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const allRanges = doc.GetAllRanges();

  const results = [];
  allRanges.forEach(function (range) {
    results.push({
      text: range.GetText(),
      start: range.GetStart(),
      end: range.GetEnd(),
    });
  });

  return results;
}, false);
```

### 选区边界

获取选区起始和结束位置的信息：

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    return {
      start: range.GetStart(),
      end: range.GetEnd(),
      length: range.GetEnd() - range.GetStart(),
    };
  }
  return null;
}, false);
```

### 扩展选区

以编程方式扩展或修改选区：

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const range = doc.GetRangeBySelect();

  if (range) {
    // Select the entire paragraph containing the selection
    const paragraph = range.GetParagraph();
    if (paragraph) {
      const paraRange = paragraph.GetRange();
      doc.Select(paraRange);
    }
  }
}, false);
```

## 实际示例

### 选区字数统计

```javascript
function countWordsInSelection() {
  window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
    if (text && text.trim()) {
      const words = text.trim().split(/\s+/).length;
      const characters = text.length;
      const charactersNoSpaces = text.replace(/\s/g, "").length;

      displayStats({
        words: words,
        characters: characters,
        charactersNoSpaces: charactersNoSpaces,
      });
    }
  });
}
```

### 文本转换工具

```javascript
function transformSelection(transformType) {
  window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
    if (!text) return;

    let transformed;
    switch (transformType) {
      case "uppercase":
        transformed = text.toUpperCase();
        break;
      case "lowercase":
        transformed = text.toLowerCase();
        break;
      case "titlecase":
        transformed = text
          .toLowerCase()
          .replace(/\b\w/g, (c) => c.toUpperCase());
        break;
      case "reverse":
        transformed = text.split("").reverse().join("");
        break;
    }

    window.Asc.plugin.executeMethod("PasteText", [transformed]);
  });
}
```

### 选区高亮工具

```javascript
function highlightSelection(color) {
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range) {
      const textPr = Api.CreateTextPr();
      textPr.SetHighlight(color);
      range.SetTextPr(textPr);
    }
  }, false);
}

// Usage
highlightSelection("yellow");
```

### 格式刷

```javascript
let copiedFormat = null;

function copyFormat() {
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range) {
      copiedFormat = range.GetTextPr();
      return "Format copied";
    }
    return "No selection";
  }, false);
}

function pasteFormat() {
  if (!copiedFormat) {
    console.log("No format copied");
    return;
  }

  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const range = doc.GetRangeBySelect();

    if (range && Asc.scope.copiedFormat) {
      range.SetTextPr(Asc.scope.copiedFormat);
    }
  }, false);
}
```

## 最佳实践

### 始终检查空选区

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (!text || text.trim() === "") {
    showMessage("Please select some text first");
    return;
  }

  // Process the selection
  processText(text);
});
```

### 向用户提供反馈

```javascript
window.Asc.plugin.executeMethod("GetSelectedText", [], function (text) {
  if (text) {
    showLoadingIndicator();
    processSelection(text).then(() => {
      hideLoadingIndicator();
      showSuccessMessage("Selection processed successfully");
    });
  }
});
```

### 优雅地处理选区变化

```javascript
let lastSelection = null;

window.Asc.plugin.attachEvent("onSelectionChanged", function (selection) {
  // Debounce to avoid excessive processing
  clearTimeout(window.selectionTimeout);

  window.selectionTimeout = setTimeout(() => {
    if (selection && selection.text !== lastSelection) {
      lastSelection = selection.text;
      updatePluginForSelection(selection.text);
    }
  }, 300);
});
```

### 在插件关闭时清理资源

```javascript
window.Asc.plugin.button = function (id) {
  if (id === -1) {
    // Clear any stored selection data
    lastSelection = null;
    copiedFormat = null;

    // Close the plugin
    window.Asc.plugin.executeCommand("close", "");
  }
};
```

## 故障排除

### 未检测到选区

**问题：** `GetSelectedText` 返回空值或 null

**解决方案：**

- 确保文档中确实有文本被选中
- 检查插件是否具有适当的权限
- 验证编辑器处于编辑模式（而非查看模式）

### 选区变化未触发事件

**问题：** `onSelectionChanged` 事件未触发

**解决方案：**

- 确认已使用 `attachEvent` 方法附加了事件
- 检查配置中 `initOnSelectionChanged` 是否设置正确
- 确保插件已正确初始化

### 替换时丢失格式

**问题：** 替换文本时丢失格式

**解决方案：**

- 对带格式的内容使用 `PasteHtml` 而非 `PasteText`
- 使用带 `SetTextPr` 的 `callCommand` 以保留格式
- 在替换前保存原始格式，并在替换后重新应用

## 结论

使用选区是创建交互式且实用的 ONLYOFFICE 插件的关键所在。通过了解如何检测、读取和操作选区，您可以构建能够无缝融入用户工作流程并增强文档编辑能力的插件。
