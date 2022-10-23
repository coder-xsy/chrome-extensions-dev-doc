# [浏览器插件结构概览](https://developer.chrome.com/docs/extensions/mv3/architecture-overview/)
浏览器插件由包含的html、css、JavaScript、图片和其他浏览器插件使用到的文件组成，他可以修改浏览器的外观表现和交互方式,也可以用来修改和拓展浏览器本身的功能    
本文重点介绍浏览器插件的组成部分及其内容、如何使用浏览器api、浏览器不同部分之间的数据交流方式和插件的数据存储   
##  插件结构    
一个浏览器插件的组成部分由他的功能所决定。其中必要的一个组成文件即使`manifest.json`,下面最常用的几部分则是由浏览器的功能决定是否使用：
- service worker
- 工具栏icon
- UI内容
- Content Script
- options page
### manifest.json
`manifest.json`文件用来定义一些浏览器插件的基本信息，他是浏览器插件的最小组成，类似web开发中的入口文件，用来给浏览器提供插件的具体信息，一个最常见的`manifest.json`文件如下所示：
```
{
  "name": "My Extension",
  "description": "A nice little demo extension.",
  "version": "2.1",
  "manifest_version": 3,
  "icons": {
    "16": "icon_16.png",
    "48": "icon_48.png",
    "128": "icon_128.png"
  },
  "background": {
    "service_worker": "background.js"
  },
  "permissions": ["storage"],
  "host_permissions": ["*://*.example.com/*"],
  "action": {
    "default_icon": "icon_16.png",
    "default_popup": "popup.html"
  }
}
```
### 工具栏icon(Toolbar icon)
最为最常见的一种浏览器插件交互形式，每一个浏览器插件都可以定义自己展示在浏览器工具栏展示的icon(如果没有自定义，会默认取插件第一个字母/文字作为icon内容);用户可以查看工具栏icon来判断自己安装过的插件，通过点击icon，我们可以给用户的点击操作添加一些交互行为，最常用形式的就是用户点击icon之后我们给用户一个弹窗展示对应的交互功能，开发者之需要在`manifest.json`中通过`popup`来配置这类交互,具体可以参考
[插件的交互设计](./design-ui-interface.md)
![toolbar icon](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/ku5Z8MMssgw6MKctpJVI.png?auto=format&w=374)
### service worker
浏览器插件的service worker用来定义浏览器插件的事件处理，在service worker中可以监听所有对于浏览器重要的事件；安装插件之后，正常service worker不会调用，只有对应的事件才会触发他的执行，理论上只要在`manifest.json`中声明了对应的权限，在service worker中可以使用所有的浏览器插件的[chrome api](./api-reference.md)
通过service worker 监听浏览器事件的方式可以参考[这里](https://developer.chrome.com/docs/extensions/mv3/service_workers/)
### Content Script
Content Script 运行插件在当前tab打开的页面中插入js逻辑，它会运行在当前打开的http页面中的上下文中执行对应的js逻辑，所以我们可以通过Content Script来获取当前页面的内容
Content Script插入当前页面中执行，也可以通过浏览器插件的[Message信息传递](https://developer.chrome.com/docs/extensions/mv3/messaging/)已经插件的[storage](https://developer.chrome.com/docs/extensions/reference/storage/)和当前浏览器插件进行交互

Content Script的具体的使用方法参考[Content scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)

### UI内容
浏览器插件可以提供插件独有的交互内容，他的设计规则应该目的明确，并且简洁明了，需要有助于浏览器本身的功能补充或优化，不应该对浏览器本身的功能产生负面影响    
最常见的交互形式包含以下8种：
- 浏览器插件icon点击事件
- 浏览器插件icon的弹窗
- 新增浏览器右键菜单项
- 输入框关键字唤起插件
- 修改/新增浏览器快捷键
- 发起桌面系统通知
- 定义语言引擎，文字转语音
- 自定义页面来覆盖浏览器页面(新开tab、收藏夹、历史记录)
  
浏览器插件支持的UI方式可以参考[插件交互方式汇总](./design-ui-interface.md)
### optoins page
options page是浏览器插件自定义的一个html页面，会包含在浏览器的打包产物中。    
他的功能就如同浏览器插件可以自定义/优化浏览器的交互体验和功能一样，插件的options page可以用来提供给用户自由配置他们需要的插件内容
用户可以直接通过打开插件options page的直达链接或者通过右键工具栏icon的菜单项中的选项入口进入options page
![link](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/Mz7GV76tFkzxRlb7Pq6e.png?auto=format&w=1600)![menu](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/BM11QeGCThsUNTlsZbAe.png?auto=format&w=714)

如何开发浏览器插件的options page 参考[定义options page](https://developer.chrome.com/docs/extensions/mv3/options/)
### 额外的的html页面
除了最常见的popup弹窗和options page外，还可以在浏览器插件中定义其他的html页面，在这些html中引入的js可以和正常的浏览器插件开发中的js一样正常使用`chrome api`(需要声明对应api的所需的插件权限), 这些页面不需要在`manifest.json`中声明，你可以通过web api`window.open`和浏览器插件的api`window.create`、`tabs.create`来直接在浏览器窗口中打开（对应地址为html对于浏览器插件的相对地址)

## 插件的其他文件
### 访问插件内部的静态资源文件
- 在浏览器插件开发中，可以引用插件本地的静态资源文件，类似图片之类,可以在开发文件中使用相对地址使用插件本地文件,如下：
  ```
  <img src="images/my_image.png">
  ```
- 但是在content Script无法和上方的方式直接引用，因为content Script是运行在当前页面的上下文中，但是我们可以使用`chrome api`来获取插件内部文件的绝对地址，如下所示：
  ```
  let image = chrome.runtime.getURL("images/my_image.png")
  ```
- 我们也可以在浏览器窗口中直接访问插件内容，就如同上面定义的[额外的的html页面](#额外的的html页面)，浏览器插件的访问链接有如下结构
  ```
  chrome-extension://EXTENSION_ID/RELATIVE_PATH
  ```
  >EXTENSION_ID: 插件的唯一id,每个浏览器插件发布之后都会有自己对应的id    
  >RELATIVE_PATH: 当前访问文件对于浏览器插件打包之后的顶层目录的相对地址 

  >注意：在本地开发时，每一次浏览器插件加载的时候都会给浏览器插件生成一个新的EXTENSION_ID，除非在`manifest.json`的时候声明了`key`

### 外部可以访问的插件文件
浏览器插件包含的html、css、javascript、image文件可以提供给外部访问（content scirpt、 web页面、其他插件）    
我们可以在`manifest.json`中定义哪些文件可以访问已经其他访问条件，只有满足条件，对应的文件才可以被外部访问
```
{
  ...
  "web_accessible_resources": [
    {
      "resources": [ "images/*.png" ],
      "matches": [ "https://example.com/*" ]
    }
  ],
  ...
}
```
具体的配置方式参考[Web Accessible Resources](https://developer.chrome.com/docs/extensions/mv3/manifest/web_accessible_resources/)

## 使用浏览器api
吃了和正常开发web一样的web api，为了丰富浏览器插件的能力，浏览器还为插件提供了很多额外和浏览器本身绑定的api，例如插件可以和正常web page一样可以使用`window.open()`来新开一个页面链接，但是插件可以使用`chrome.tabs.create`这个api来选择打开链接的tab
更多的浏览器插件api可以参考[浏览器插件api汇总](./api-reference.md)

浏览器提供给插件的api包含异步方法和同步方式
### 异步方法
插件的api大多数都是异步方法，同时支持callback和promise两种调用方式，api的执行结果可以通过会作为参数传递给callback和promise.resolve    
如下所示：
```
// callback
chrome.tabs.query(queryOptions, function(tabs) {
  chrome.tabs.update(tabs[0].id, {url: newUrl});
});
someOtherFunction();
```
```
// Promise
chrome.tabs.query(queryOptions)
.then((tabs) => {
  chrome.tabs.update(tabs[0].id, {url: newUrl});
  someOtherFunction();
});

// async-await
async function queryTab() {
  let tabs = await chrome.tabs.query(queryOptions);
  chrome.tabs.update(tabs[0].id, {url: newUrl});
  someOtherFunction();
}
```
> 在V3的浏览器插件中，大多数异步方法都支持了promise调用
> 但是也需要注意，不是所有的api都支持，可以查看对应的api介绍来确认
### 同步方法
当然插件api也有同步方法，如下：
```
// Synchronous methods have no callback
const imgUrl = chrome.runtime.getURL("images/icon.png")
```

## 插件不同部分的信息传递
插件可以包含很多页面，在插件的不同组成部分有时候需要共享数据，所以浏览器给一个插件的不同部分提供了一个信息传递方式[message passing](https://developer.chrome.com/docs/extensions/mv3/messaging/),这样每个部分都可以订阅信息并且做出响应

## 数据存储
为了满足浏览器插件对于数据存储的需求，浏览器提供了改进之后的`storage api`让插件可以存储数据,例如你还可以`onChange`监听storage中的数据改动,除了storage作为数据储存之外，插件还可以和web page一样使用[indexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)    
插件的storage api使用方式可以参考[storage](https://developer.chrome.com/docs/extensions/reference/storage/)
