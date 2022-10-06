# Getting started
插件采用传统的web开发方式：HTML CSS和Javscript开发，其中一个浏览器插件包含[background scripts](https://developer.chrome.com/docs/extensions/mv3/service_workers/)、[content scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)、[options page](https://developer.chrome.com/docs/extensions/mv3/options/)、[UI elements](https://developer.chrome.com/docs/extensions/mv3/user_interface/)和其他逻辑文件，这些内容取决于浏览器插件的具体功能可以做一些取舍    
本文将帮助你开发一个允许用户修改浏览器当前tab页面的背景颜色的浏览器插件，其中会使用上述提到的浏览器插件内容来帮助你理解其中各个模块的关系   
开始之前，你需要创建一个新的开发目录，我们默认在该目录下进行浏览器插件的开发
一个完成的浏览器插件demo你可以在这里[下载](https://storage.googleapis.com/web-dev-uploads/file/WlD8wC6g8khYWPJUsQceQkhXSlv1/SVxMBoc5P3f6YV3O7Xbu.zip)
## 创建一个manifest文件
在目录下创建一个manifest.json文件，内容如下,这里的manifest_version代表浏览器的的开发版本，我们默认使用V3版本开发，如果你使用的是V2版本，你需要设置为2     
这是一个浏览器插件最基础的组成内容
```
{
  "name": "Getting Started Example",
  "description": "Build an Extension!",
  "version": "1.0",
  "manifest_version": 3
}
```
##  安装开发的浏览器插件
创建好manifest.json文件之后当前目录已经可以当作一个基础的浏览器插件被安装到浏览器中，可以按照下方的步骤进行安装   
> 1.  打开浏览器的插件管理页面（可以直接浏览器访问当前页面：`chrome://extensions`）
> 2.  打开插件管理页面的开发者模式
> 3.  点击加载解压的浏览器插件的按钮，然后选择我们开发目录            
  
当看到下面的效果，代表安装成功<br>
![插件安装示例](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/vOu7iPbaapkALed96rzN.png?auto=format)

## 为浏览器添加逻辑
现在浏览器已经被安装成功了，但是我们还没有添加任何逻辑所有它还不包含任何功能。现在我们添加一些代码来实现存储一个颜色值的功能
### 在manifest文件中添加background script 声明
在manifest.json文件中声明的background script的js文件用来定义浏览器插件在后台静默执行的时候的动作行为，在manifest.json文件中添加如下代码
```
{
  ...
  "background": {
    "service_worker": "background.js"
  }
}
```
当你重新加载浏览器插件时，浏览器会自动扫描manifest.json文件，自动识别并执行声明的background script对应的js文件
### 创建一个background script文件
上一步已经声明了background script文件，现在我们需要创建对应的`background.js`文件，在文件中我们通过api`runtime.onInstalled`来监听浏览器插件安装的生命周期，在监听事件中我们在storage中设置一个value;具体的代码如下
```
// background.js

let color = '#3aa757';

chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.sync.set({ color });
  console.log('Default background color set to %cgreen', `color: ${color}`);
});
```
### 添加storage权限声明
在浏览器开发中，很多api的使用需要声明一些用户授权，这些权限就需要在manifest.json文件中提前声明，其中我们在background.js文件中使用到的`chrome.storage`就需要使用到stroage权限，所以我们需要在manifest.json中添加如下代码
```
{
  ...
  "permissions": ["storage"]
}
```
在重新加载浏览器插件之后，浏览器管理页面如下，在我们的浏览器卡片中会新增一行 Inspect views service worker，如下：<br>
![声明 backgournd script](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/dx9EpIKK949olhe8qraK.png?auto=format)
点击service worker的链接之后可以出现浏览器的调试控制台，对于background script 的调试内容，可以看到log日志：`Default background color set to green`

## 插件和用户的交互
浏览器插件和用户的交互可以有很多种方式，其中一种就是通过浏览器的popup小窗口    
接下来我们将实现一个popup窗口功能，并结合之前的步骤内容，实现用户可以通过窗口中的按钮修改背景颜色的功能   
开发方式需要在项目目录下新建一个`popup.html`文件，copy以下内容到html文件中
```
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="button.css">
  </head>
  <body>
    <button id="changeColor"></button>
  </body
</html>
```
和background script一样，我们需要在manifest.json文件中声明我们希望使用的html文件作为弹出窗口内容，manifest.json文件中添加如下内容,action表示我们希望用户在浏览器上点击插件icon的动作表现，我们声明为弹出一个窗口并且内容有`popup.html`定义
```
{
  "action": {
    "default_popup": "popup.html"
  }
}
```
我们可以看到html文件中还引入了对应的`button.css`的样式文件，所以我们在插件根目录下添加该文件，并且添加如下内容至文件中
```
button {
  height: 30px;
  width: 30px;
  outline: none;
  margin: 10px;
  border: none;
  border-radius: 2px;
}

button.current {
  box-shadow: 0 0 0 2px white,
              0 0 0 4px black;
}
```
至此我们定义了弹出窗口的内容，但是有时候我们希望我们自己开发的浏览器插件在用户的浏览器工具栏展示的icon是自己定义的icon，我们可以通过在`manifest.json`文件中`action.default_icon`来自定义自己的插件的icon（自行安排图片文件目录），如下代码
```
{
  ...
  "action": {
    ...
    "default_icon": {
      "16": "/images/get_started16.png",
      "32": "/images/get_started32.png",
      "48": "/images/get_started48.png",
      "128": "/images/get_started128.png"
    }
  }
}
```
`action.default_icon`设置的是浏览器插件在浏览器工具栏展示的内容，同时我们也可以通过`icons`来设置浏览器插件卡片在浏览器插件管理页面展示的icon内容,可以在`manifest.json`文件中添加如下代码：
```
{
  ...
  "icons": {
    "16": "/images/get_started16.png",
    "32": "/images/get_started32.png",
    "48": "/images/get_started48.png",
    "128": "/images/get_started128.png"
  }
}
```
设置完成并且重新加载浏览器插件之后可以发现浏览器插件icon并没有展示在浏览器的工具栏，因为浏览器插件安装之后默认不会固定展示在工具栏右侧，需要手动设置固定展示，如下图：
![固定浏览器插件](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/GdHNy255kS4hWD5vb1fc.png?auto=format)   
之后我们就可以点击浏览器工具栏对应的插件icon,出来我们自定义的popup弹窗页面内容，如下图：    
![popup页面](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/ku5Z8MMssgw6MKctpJVI.png?auto=format)      
和普通的web开发一样，目前popup页面中只包含静态内容，并没有任何交互，我们可以通过引入js文件来添加交互    
我们在`popup.html`同级目录添加`popup.js`，并且添加如下带内容：
```
// Initialize button with user's preferred color
let changeColor = document.getElementById("changeColor");

chrome.storage.sync.get("color", ({ color }) => {
  changeColor.style.backgroundColor = color;
});
```
这段js代码为popup窗口中的按钮添加了一个背景色，其中背景色的取值来之storage    
我们将这个js文件引入到`popup.html`文件中,`popup.html`文件如下：
```
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="button.css">
  </head>
  <body>
    <button id="changeColor"></button>
    <script src="popup.js"></script>
  </body>
</html>
```
重新加载浏览器插件之后我们重新打开popup窗口，发现按钮已经变成了绿色了（我们在之前的`background.js`文件中设置了storage中的value）
## 添加更多逻辑
我们当前设置了自定义的icon和popup窗口内容，并且添加了一段逻辑实现了popup窗口中的按钮背景颜色为浏览器storage中设置的颜色值，现在我们在`popup.js`中添加更多的逻辑来增强用户交互，追加如下代码：
```
// When the button is clicked, inject setPageBackgroundColor into current page
changeColor.addEventListener("click", async () => {
  let [tab] = await chrome.tabs.query({ active: true, currentWindow: true });

  chrome.scripting.executeScript({
    target: { tabId: tab.id },
    func: setPageBackgroundColor,
  });
});

// The body of this function will be executed as a content script inside the
// current page
function setPageBackgroundColor() {
  chrome.storage.sync.get("color", ({ color }) => {
    document.body.style.backgroundColor = color;
  });
}
```
上述的代码为popup窗口中的按钮新增了一个点击监听事件，用户点击按钮之后在对应的tab中执行一段[`content script`](https://developer.chrome.com/docs/extensions/mv3/content_scripts/#programmatic),它实现了将当前窗口的页面中的背景色设置为和按钮背景色一致的颜色，动态选择code执行的窗口环境可以让用户自由选择当前的js逻辑执行的tab窗口    
在上面一段的代码中使用了`chrome.tabs.query`（获取当前浏览器打开的tab窗口）和`chrome.scripting.executeScript`（执行动态插入的js逻辑）这两个api,所以在storage权限之后，需要额外的声明两个权限:[activeTab](https://developer.chrome.com/docs/extensions/mv3/manifest/activeTab/)、[scripting](https://developer.chrome.com/docs/extensions/reference/scripting/)
```
{
  "name": "Getting Started Example",
  ...
  "permissions": ["storage", "activeTab", "scripting"],
  ...
}
```
重新加载浏览器插件之后，打开popup窗口点击按钮，发现当前的浏览器窗口背景色变成了我们设置的绿色   
接下来我们将实现用户可以选择设置不同颜色的功能    

## 设置一个用户操作页面
接下来我们将实现用户可以选择设置不同颜色的功能    
在项目根目录下创建一个`options.html`文件，包含如下内容：
```
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="button.css">
  </head>
  <body>
    <div id="buttonDiv">
    </div>
    <div>
      <p>Choose a different background color!</p>
    </div>
    <script src="options.js"></script>
  </body>
</html>
```
和之前一样，我们需要在`manifest.json`文件中声明options page的定义文件，如下：
```
{
  "name": "Getting Started Example",
  ...
  "options_page": "options.html"
}
```
重新加载浏览器插件之后，我们右键点击浏览器工具栏中的浏览器插件icon,选择options选项，它会自动在浏览器窗口中打开我们声明的`options_page`对应的html页面，如下图所示：
![打开options页面](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/aV46PP8KCjEqqenSJxxp.png?auto=format)
和之前定义popup页面类似，我们在`options.html`中引入了`options.js`文件来添加交互逻辑，所以我们在项目根目录下新增该文件，并添加如下内容：
```
let page = document.getElementById("buttonDiv");
let selectedClassName = "current";
const presetButtonColors = ["#3aa757", "#e8453c", "#f9bb2d", "#4688f1"];

// Reacts to a button click by marking the selected button and saving
// the selection
function handleButtonClick(event) {
  // Remove styling from the previously selected color
  let current = event.target.parentElement.querySelector(
    `.${selectedClassName}`
  );
  if (current && current !== event.target) {
    current.classList.remove(selectedClassName);
  }

  // Mark the button as selected
  let color = event.target.dataset.color;
  event.target.classList.add(selectedClassName);
  chrome.storage.sync.set({ color });
}

// Add a button to the page for each supplied color
function constructOptions(buttonColors) {
  chrome.storage.sync.get("color", (data) => {
    let currentColor = data.color;
    // For each color we were provided…
    for (let buttonColor of buttonColors) {
      // …create a button with that color…
      let button = document.createElement("button");
      button.dataset.color = buttonColor;
      button.style.backgroundColor = buttonColor;

      // …mark the currently selected color…
      if (buttonColor === currentColor) {
        button.classList.add(selectedClassName);
      }

      // …and register a listener for when that button is clicked
      button.addEventListener("click", handleButtonClick);
      page.appendChild(button);
    }
  });
}

// Initialize the page by constructing the color options
constructOptions(presetButtonColors);
```
我们在options页面中为用户添加了四个待选择的颜色，用户可以选中对应的颜色，然后点击页面中的按钮，它会自动更新浏览器插件中缓存的颜色色值，然后没有用户通过点击popup页面中的按钮设置的页面背景色都会和我们选择的背景色保持一致

> <strong>至此我们已经亲手搭建了一个具有一定功能的Chrome浏览器插件<strong>
