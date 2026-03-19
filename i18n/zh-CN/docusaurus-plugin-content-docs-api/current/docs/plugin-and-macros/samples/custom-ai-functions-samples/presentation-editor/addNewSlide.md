# addNewSlide

该函数使用当前幻灯片母版中的默认版式，在演示文稿末尾添加一张新幻灯片。

## 提示词

- 添加一张新幻灯片

## 函数注册 {#function-registration}

```ts
(function () {
  let func = new RegisteredFunction({
    name: "addNewSlide",
    description:
      "Adds a new slide at the end of presentation using default layout from current slide's master",
    parameters: {
      type: "object",
      properties: {},
      required: [],
    },
    examples: [
      {
        prompt: "Add new slide",
        arguments: {},
      },
    ],
  });

  return func;
})();
```

## 函数执行 {#function-execution}

```ts
func.call = async function (params) {
  Asc.scope.params = params;

  await Asc.Editor.callCommand(function () {
    let presentation = Api.GetPresentation();
    let currentSlide = presentation.GetCurrentSlide();
    let master;

    if (!currentSlide) {
      currentSlide = presentation.GetSlideByIndex(0);
      let curLayout = currentSlide.GetLayout();
      master = curLayout.GetMaster();
    } else {
      master = presentation.GetMasterByIndex(0);
    }

    if (!master) return;

    let layout = master.GetLayoutByType("obj");
    if (!layout) {
      let layoutsCount = master.GetLayoutsCount();
      if (layoutsCount > 0) {
        layout = master.GetLayout(0);
      }
    }

    if (!layout) return;

    let newSlide = Api.CreateSlide();
    newSlide.ApplyLayout(layout);
    presentation.AddSlide(newSlide);
  });
};
```

使用的方法：[GetPresentation](/docs/office-api/usage-api/presentation-api/Api/Methods/GetPresentation.md), [GetCurrentSlide](/docs/office-api/usage-api/presentation-api/ApiPresentation/Methods/GetCurrentSlide.md), [GetSlideByIndex](/docs/office-api/usage-api/presentation-api/ApiPresentation/Methods/GetSlideByIndex.md), [GetLayout](/docs/office-api/usage-api/presentation-api/ApiSlide/Methods/GetLayout.md), [GetMaster](/docs/office-api/usage-api/presentation-api/ApiLayout/Methods/GetMaster.md), [GetLayoutByType](/docs/office-api/usage-api/presentation-api/ApiMaster/Methods/GetLayoutByType.md), [GetLayoutsCount](/docs/office-api/usage-api/presentation-api/ApiMaster/Methods/GetLayoutsCount.md), [GetLayout](/docs/office-api/usage-api/presentation-api/ApiMaster/Methods/GetLayout.md), [CreateSlide](/docs/office-api/usage-api/presentation-api/Api/Methods/CreateSlide.md), [ApplyLayout](/docs/office-api/usage-api/presentation-api/ApiSlide/Methods/ApplyLayout.md), [AddSlide](/docs/office-api/usage-api/presentation-api/ApiPresentation/Methods/AddSlide.md)

## 结果

<video className="light-video" controls style={{maxWidth: '848px'}}>

  <source src="/assets/images/plugins/functions-video/presentation-editor/addNewSlide.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
<video className="dark-video" controls style={{maxWidth: '848px'}}>
  <source src="/assets/images/plugins/functions-video/presentation-editor/addNewSlide.dark.webm" type="video/webm" />
  您的浏览器不支持 HTML5 视频。
</video>
