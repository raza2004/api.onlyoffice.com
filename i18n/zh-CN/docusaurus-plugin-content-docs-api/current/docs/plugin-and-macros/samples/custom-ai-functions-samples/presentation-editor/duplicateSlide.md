# duplicateSlide

该函数复制当前演示文稿中的指定幻灯片。

## 提示词

- 复制第 8 张幻灯片

## 函数注册 {#function-registration}

```ts
(function () {
  let func = new RegisteredFunction({
    name: "duplicateSlide",
    description: "Duplicates slide with the specific index or current",
    parameters: {
      type: "object",
      properties: {
        slideNumber: {
          type: "number",
          description: "the slide number to duplicate",
          minimum: 1,
        },
      },
      required: [],
    },
    examples: [
      {
        prompt: "duplicate slide 3",
        arguments: { slideNumber: 3 },
      },

      {
        prompt: "duplicate slide",
        arguments: {},
      },
    ],
  });
  return func;
})();
```

### 参数

| 名称        | 类型   | 示例 | 描述                   |
| ----------- | ------ | ---- | ---------------------- |
| slideNumber | number | 2    | 要复制的幻灯片编号。   |

## 函数执行 {#function-execution}

```ts
func.call = async function (params) {
  Asc.scope.slideNum = params.slideNumber;
  let data = await Asc.Editor.callCommand(function () {
    let presentation = Api.GetPresentation();
    let slide;
    if (Asc.scope.slideNum !== undefined && Asc.scope.slideNum !== null) {
      slide = presentation.GetSlideByIndex(Asc.scope.slideNum - 1);
    }
    if (!slide) slide = presentation.GetCurrentSlide();
    if (!slide) {
      return null;
    }
    let slideIdx = slide.GetSlideIndex();
    if (slide) {
      slide.Duplicate(slideIdx + 1);
      return { idx: slideIdx + 1 };
    }
    return null;
  });
  if (data) {
    await Asc.Editor.callMethod("GoToSlide", [data["idx"] + 1]);
  }
};
```

使用的方法：[GetPresentation](/docs/office-api/usage-api/presentation-api/Api/Methods/GetPresentation.md), [GetSlideByIndex](/docs/office-api/usage-api/presentation-api/ApiPresentation/Methods/GetSlideByIndex.md), [Duplicate](/docs/office-api/usage-api/presentation-api/ApiSlide/Methods/Duplicate.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/presentation-editor/duplicateSlide.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/presentation-editor/duplicateSlide.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
