# deleteSlide

该函数从当前演示文稿中删除指定幻灯片。

## 提示词

- 删除第 8 张幻灯片

## 函数注册 {#function-registration}

```ts
(function () {
  let func = new RegisteredFunction({
    name: "deleteSlide",
    description: "Deletes slide with the specific index or current",
    parameters: {
      type: "object",
      properties: {
        slideNumber: {
          type: "number",
          description: "the slide number to delete",
          minimum: 1,
        },
      },
      required: [],
    },
    examples: [
      {
        prompt: "delete slide 5",
        arguments: { slideNumber: 5 },
      },

      {
        prompt: "delete slide",
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
| slideNumber | number | 2    | 要删除的幻灯片编号。   |

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
    let curSlideIdx = presentation.GetCurSlideIndex();
    let slideIdx = slide.GetSlideIndex();
    slide.Delete();
    return { curSlideIdx: curSlideIdx, slideIdx: slideIdx };
  });
  if (data) {
    if (data["slideIdx"] <= data["curSlideIdx"]) {
      await Asc.Editor.callMethod("GoToSlide", [data["curSlideIdx"]]);
    }
  }
};
```

使用的方法：[GetPresentation](/docs/office-api/usage-api/presentation-api/Api/Methods/GetPresentation.md), [GetSlideByIndex](/docs/office-api/usage-api/presentation-api/ApiPresentation/Methods/GetSlideByIndex.md), [Delete](/docs/office-api/usage-api/presentation-api/ApiSlide/Methods/Delete.md), [Asc.scope object](/docs/plugin-and-macros/interacting-with-editors/overview/how-to-call-commands.md#ascscope-object)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/presentation-editor/deleteSlide.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/presentation-editor/deleteSlide.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
