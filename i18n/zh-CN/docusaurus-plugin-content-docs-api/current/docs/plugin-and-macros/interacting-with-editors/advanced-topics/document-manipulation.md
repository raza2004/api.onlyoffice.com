---
sidebar_position: -3
---

# 文档操作

## 概述

文档操作允许插件以编程方式创建、修改和转换文档内容。本指南涵盖了在所有编辑器类型中使用 ONLYOFFICE 文档的高级技术。

## 了解文档结构

### 文档层次结构

ONLYOFFICE 文档遵循层次结构：

```
Document
├── Sections
│   ├── Paragraphs
│   │   ├── Runs (text with formatting)
│   │   └── Inline elements (images, hyperlinks)
│   └── Tables
│       ├── Rows
│       └── Cells
└── Styles
```

### 访问文档

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Get document properties
  const paragraphs = doc.GetAllParagraphs();
  const tables = doc.GetAllTables();

  return {
    paragraphCount: paragraphs.length,
    tableCount: tables.length,
  };
}, false);
```

## 创建内容

### 添加段落

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Create a new paragraph
  const para = Api.CreateParagraph();
  para.AddText("This is a new paragraph");

  // Push to document
  doc.Push(para);
}, false);
```

### 创建格式化文本

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const para = Api.CreateParagraph();

  // Create text with formatting
  const run = para.AddText("Bold and Italic Text");
  run.SetBold(true);
  run.SetItalic(true);
  run.SetFontSize(16);
  run.SetColor(255, 0, 0, false); // Red color

  doc.Push(para);
}, false);
```

### 插入表格

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Create table (3 rows, 4 columns)
  const table = Api.CreateTable(4, 3);

  // Fill table cells
  for (let row = 0; row < 3; row++) {
    for (let col = 0; col < 4; col++) {
      const cell = table.GetCell(row, col);
      const cellContent = cell.GetContent();
      const para = cellContent.GetElement(0);
      para.AddText(`Cell ${row}-${col}`);
    }
  }

  doc.Push(table);
}, false);
```

## 修改现有内容

### 查找和替换文本

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Search for text
  const searchTerm = "old text";
  const replaceTerm = "new text";

  const results = doc.Search(searchTerm, true); // true = case-insensitive

  // Replace each occurrence
  results.forEach(function (range) {
    range.Delete();
    range.AddText(replaceTerm);
  });

  return results.length;
}, false);
```

### 修改段落

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();

  // Modify first paragraph
  if (paragraphs.length > 0) {
    const firstPara = paragraphs[0];

    // Set paragraph properties
    firstPara.SetJc("center"); // Align center
    firstPara.SetSpacingBefore(240); // Spacing before (in twips)
    firstPara.SetSpacingAfter(240); // Spacing after

    // Apply text formatting
    const textPr = Api.CreateTextPr();
    textPr.SetBold(true);
    textPr.SetFontSize(18);
    firstPara.SetTextPr(textPr);
  }
}, false);
```

### 使用表格

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const tables = doc.GetAllTables();

  if (tables.length > 0) {
    const table = tables[0];

    // Add a new row
    table.AddRow(table.GetRowsCount());

    // Merge cells
    const cell1 = table.GetCell(0, 0);
    const cell2 = table.GetCell(0, 1);
    cell1.Merge(cell2);

    // Set cell background color
    cell1.SetShd("clear", 200, 200, 255, false);
  }
}, false);
```

## 高级文档操作

### 创建样式

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Create a custom style
  const style = doc.GetStyle("Heading 1");
  const customStyle = doc.CreateStyle("CustomHeading", "paragraph");

  // Copy properties from existing style
  customStyle.SetBasedOn(style);

  // Customize
  const textPr = customStyle.GetTextPr();
  textPr.SetColor(0, 100, 200, false);
  textPr.SetFontSize(20);

  // Apply to paragraph
  const para = Api.CreateParagraph();
  para.AddText("Custom Styled Heading");
  para.SetStyle(customStyle);

  doc.Push(para);
}, false);
```

### 添加页眉和页脚

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const sections = doc.GetSections();

  if (sections.length > 0) {
    const section = sections[0];

    // Add header
    const header = section.GetHeader("default", true);
    const headerPara = header.GetElement(0);
    headerPara.AddText("Document Header");
    headerPara.SetJc("center");

    // Add footer
    const footer = section.GetFooter("default", true);
    const footerPara = footer.GetElement(0);
    footerPara.AddText("Page ");
    footerPara.AddPageNumber();
  }
}, false);
```

### 插入图片

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();

  // Create paragraph for image
  const para = Api.CreateParagraph();

  // Add image (base64 or URL)
  const imageUrl = "https://example.com/image.png";
  const drawing = Api.CreateImage(imageUrl, 200 * 36000, 150 * 36000);

  para.AddDrawing(drawing);
  doc.Push(para);
}, false);
```

## 批量操作

### 处理所有段落

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();

  let processedCount = 0;

  paragraphs.forEach(function (para) {
    const text = para.GetText();

    // Skip empty paragraphs
    if (!text || text.trim() === "") return;

    // Process paragraph
    if (text.length > 100) {
      // Highlight long paragraphs
      const textPr = Api.CreateTextPr();
      textPr.SetHighlight("yellow");
      para.SetTextPr(textPr);
      processedCount++;
    }
  });

  return processedCount;
}, false);
```

### 批量格式化

```javascript
function applyBulkFormatting(options) {
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    const paragraphs = doc.GetAllParagraphs();

    const textPr = Api.CreateTextPr();

    if (Asc.scope.options.fontFamily) {
      textPr.SetFontFamily(Asc.scope.options.fontFamily);
    }
    if (Asc.scope.options.fontSize) {
      textPr.SetFontSize(Asc.scope.options.fontSize);
    }
    if (Asc.scope.options.color) {
      const rgb = Asc.scope.options.color;
      textPr.SetColor(rgb.r, rgb.g, rgb.b, false);
    }

    paragraphs.forEach(function (para) {
      para.SetTextPr(textPr);
    });
  }, false);

  // Store options in Asc.scope for access in callCommand
  Asc.scope.options = options;
}

// Usage
applyBulkFormatting({
  fontFamily: "Arial",
  fontSize: 12,
  color: { r: 0, g: 0, b: 0 },
});
```

## 文档分析

### 提取文档统计信息

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();
  const tables = doc.GetAllTables();

  let wordCount = 0;
  let charCount = 0;

  paragraphs.forEach(function (para) {
    const text = para.GetText();
    if (text) {
      wordCount += text.trim().split(/\s+/).length;
      charCount += text.length;
    }
  });

  return {
    paragraphs: paragraphs.length,
    tables: tables.length,
    words: wordCount,
    characters: charCount,
  };
}, false);
```

### 查找特定内容

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();

  const results = [];

  paragraphs.forEach(function (para, index) {
    const text = para.GetText();

    // Find paragraphs containing URLs
    const urlRegex = /https?:\/\/[^\s]+/g;
    const urls = text.match(urlRegex);

    if (urls && urls.length > 0) {
      results.push({
        paragraph: index,
        urls: urls,
      });
    }
  });

  return results;
}, false);
```

## 电子表格操作

### 使用单元格

```javascript
window.Asc.plugin.callCommand(function () {
  const sheet = Api.GetActiveSheet();

  // Set cell values
  sheet.GetRange("A1").SetValue("Product");
  sheet.GetRange("B1").SetValue("Price");
  sheet.GetRange("C1").SetValue("Quantity");

  // Add data rows
  sheet.GetRange("A2").SetValue("Widget");
  sheet.GetRange("B2").SetValue(19.99);
  sheet.GetRange("C2").SetValue(5);

  // Format header row
  const headerRange = sheet.GetRange("A1:C1");
  headerRange.SetBold(true);
  headerRange.SetFillColor(Api.CreateColorFromRGB(200, 200, 255));
}, false);
```

### 添加公式

```javascript
window.Asc.plugin.callCommand(function () {
  const sheet = Api.GetActiveSheet();

  // Add SUM formula
  sheet.GetRange("D2").SetValue("=B2*C2");

  // Add total
  sheet.GetRange("D10").SetValue("=SUM(D2:D9)");

  // Format as currency
  sheet.GetRange("B2:B10").SetNumberFormat("$#,##0.00");
}, false);
```

## 演示文稿操作

### 创建幻灯片

```javascript
window.Asc.plugin.callCommand(function () {
  const presentation = Api.GetPresentation();

  // Create new slide
  const slide = Api.CreateSlide();

  // Add title
  const title = slide.GetAllShapes()[0];
  const titleContent = title.GetDocContent();
  const titlePara = titleContent.GetElement(0);
  titlePara.AddText("New Slide Title");

  // Add content
  const content = slide.GetAllShapes()[1];
  const contentDoc = content.GetDocContent();
  const contentPara = contentDoc.GetElement(0);
  contentPara.AddText("Slide content goes here");

  // Add to presentation
  presentation.AddSlide(slide);
}, false);
```

### 添加形状

```javascript
window.Asc.plugin.callCommand(function () {
  const presentation = Api.GetPresentation();
  const slide = presentation.GetCurrentSlide();

  // Create shape
  const shape = Api.CreateShape("rect", 5000000, 3000000);
  shape.SetPosition(1000000, 2000000);

  // Add text to shape
  const docContent = shape.GetDocContent();
  const para = docContent.GetElement(0);
  para.AddText("Shape with text");

  // Style the shape
  const fill = Api.CreateSolidFill(Api.CreateRGBColor(100, 150, 255));
  shape.SetFill(fill);

  // Add to slide
  slide.AddObject(shape);
}, false);
```

## 最佳实践

### 使用高效操作

**错误名称：** 在循环中执行多次文档操作

:::warning[Wrong]

```javascript
paragraphs.forEach(function (para) {
  window.Asc.plugin.callCommand(function () {
    // Calling API for each paragraph
  }, false);
});
```

:::

:::tip[Correct]

```javascript
window.Asc.plugin.callCommand(function () {
  paragraphs.forEach(function (para) {
    // Process all paragraphs in one call
  });
}, false);
```

:::

错误输出：性能低下——多次 API 调用会显著降低执行速度。

### 操作前进行验证

**错误名称：** 处理前缺少验证

:::warning[Wrong]

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();

  // Process without checking
  paragraphs.forEach(function (para) {
    para.SetTextPr(textPr);
  });
}, false);
```

:::

:::tip[Correct]

```javascript
window.Asc.plugin.callCommand(function () {
  const doc = Api.GetDocument();
  const paragraphs = doc.GetAllParagraphs();

  if (!paragraphs || paragraphs.length === 0) {
    return "No paragraphs found";
  }

  // Safe to proceed
  paragraphs.forEach(function (para) {
    // Process paragraphs
  });
}, false);
```

:::

错误输出：运行时错误——当文档为空时出现"Cannot read property 'forEach' of undefined"。

### 为长时间操作提供反馈

**错误名称：** 处理过程中无用户反馈

:::warning[Wrong]

```javascript
function processLargeDocument() {
  // No feedback - user doesn't know what's happening
  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    // Long operation that takes time
    return "Complete";
  }, false);
}
```

:::

:::tip[Correct]

```javascript
function processLargeDocument() {
  showProgressIndicator("Processing document...");

  window.Asc.plugin.callCommand(function () {
    const doc = Api.GetDocument();
    // Long operation
    return "Complete";
  }, false);
}

window.Asc.plugin.onCommandCallback = function (result) {
  hideProgressIndicator();
  showMessage("Document processed: " + result);
};
```

:::

错误输出：用户体验问题——插件看似卡死，用户可能认为已崩溃。

## 结论

文档操作是一项强大的功能，它使插件能够自动执行复杂任务并增强文档编辑工作流。通过了解文档结构和 API 方法，您可以创建插件，改变用户使用 ONLYOFFICE 文档的方式。
