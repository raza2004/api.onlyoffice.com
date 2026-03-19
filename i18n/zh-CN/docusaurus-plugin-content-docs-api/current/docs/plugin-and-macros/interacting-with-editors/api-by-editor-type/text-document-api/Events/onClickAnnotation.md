# onClickAnnotation

用户单击注释时调用的函数。

## 参数

| **名称** | **数据类型** | **描述** |
| --------- | ------------- | ----------- |
| annotation | TextAnnotation | 被单击的注释。 |

```javascript
window.Asc.plugin.attachEditorEvent("onClickAnnotation", (data) => {
    console.log("Annotation clicked:", data.rangeId);
});
```
