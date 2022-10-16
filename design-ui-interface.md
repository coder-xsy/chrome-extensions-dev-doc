# 浏览器插件交互方式设计
## 1.浏览器导航栏icon交互
定义浏览器icon的交互首选需要在`manifestion.json`的`action`这个可以下进行声明配置    
- <strong>定义浏览器工具栏的icon</strong>
  通过`action.default_icon`配置，其中`16`、`32`等，以16的倍数递增，以适配不同分辨率的屏幕;对于配置的icon要求都是正方形，如果不满足会存在截取；如果没有配置`default_icon`,会默认使用浏览器名称的第一个字母展示在浏览器icon的位置
   ```
    {
      "name": "My Awesome Extension",
      ...
      "action": {
        "default_icon": {
          "16": "extension_toolbar_icon16.png",
          "32": "extension_toolbar_icon32.png"
        }
      }
      ...
    }
   ```
  配置浏览器插件其他位置的插件:
  通过`manifest.json`下的`icons`这个key配置浏览器插件其他位置展示的icon图片
  ```
  {
    "name": "My Awesome Extension",
    ...
    "icons": {
      "16": "extension_icon16.png", // 插件的options页面和右选栏中的icon
      "32": "extension_icon32.png", // window 要求的icon
      "48": "extension_icon48.png", // 展示在插件管理页面的icon
      "128": "extension_icon128.png" // 展示在chrome插件商店的icon
    }
    ...
  }
  ```
- <strong>浏览器工具栏插件icon的active和disable状态切换</strong>
  使用浏览器插件的api`chrome.declarativeContent`定义特定的规则声明，当前窗口的url、css链接、js链接符合要求时，浏览器工具栏的浏览器插件处于正常active状态，当不符合规则时icon转变成置灰状态，(如果浏览器处于置灰状态时，点击icon会出现选项菜单，如下图)
  ![declarativeContent](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/hlYsQJPFsF7WBAjJZ6DS.png?auto=format&w=504)
- <strong>为浏览器插件icon动态添加角标</strong>   
  当浏览器插件声明了工具栏右侧的icon时，我们可以使用`chrome.action.setBadgeText`和`chrome.action.setBadgeBackgroundColor`给icon添加角标文案和角标背景色,如下所示：
  ```
  chrome.action.setBadgeText({text: 'ON'});
  chrome.action.setBadgeBackgroundColor({color: '#4688F1'});
  ```
  效果如下所示：    
  ![text](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/nXwAHSWLBEgT8099ITT0.png?auto=format&w=144)
  ![normal](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/pNz8UgfTBMmcf7fE9wja.png?auto=format&w=144)
- <strong>为浏览器插件icon添加弹出窗口</strong>   
  最常见的浏览器插件交互形式，当用户点击工具栏右侧插件icon时，出一个弹窗，我们可以自定义弹窗内容来提供用户交互和对应的功能;它就如同一个常见的web页面一般，可以使用html、css、js开发，由于运行在浏览器插件环境下，js可以直接调用浏览器插件的api,如下示例所示：
  ![water demo](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/JVduBMXnyUorfNjFZmue.png?auto=format&w=426)
  ```
  // 编写popup.html内容,这里需要注意，不能使用
  <html>
    <head>
      <title>Water Popup</title>
      <link rel="stylesheet" href="popup.css" />
    </head>
    <body>
        <img src="./stay_hydrated.png" id="hydrateImage">
        <button id="sampleSecond" value="0.1">Sample Second</button>
        <button id="min15" value="15">15 Minutes</button>
        <button id="min30" value="30">30 Minutes</button>
        <button id="cancelAlarm">Cancel Alarm</button>
      <script src="popup.js"></script>
    </body>
  </html>
  ```
  ```
  // popup.css
  .title {
    color: #ff0000;
  }
  ```
  ```
  // .popup.js 
  console.log('test')
  ```
  还需要在`manifest.json`中设置弹窗默认对应的html
  ```
  {
    "name": "Drink Water Event",
    ...
    "action": {
      "default_popup": "popup.html"
    }
    ...
  }
  ```
  <strong>同时我们可以使用`chrome.action.setPopup`在运行时动态设置弹窗对应的html文件</strong>，如下所示
  ```
  chrome.storage.local.get('signed_in', (data) => {
    if (data.signed_in) {
      chrome.action.setPopup({popup: 'popup.html'});
    } else {
      chrome.action.setPopup({popup: 'popup_sign_in.html'});
    }
  });
  ```

- <strong>添加hover提示</strong>    
  可以通过在`manifest.json`中配置`action.default_title`来设置用户hover在工具栏上的浏览器插件icon的提示文案；同时也支持通过`chrome.action.setTitle`api来动态设置
  ```
  // manifest.json
  {
    "name": "Tab Flipper",
    ...
    "action": {
      "default_title": "Press Ctrl(Win)/Command(Mac)+Shift+Right/Left to flip tabs"
    }
  ...
  }
  ```
- <strong>添加点击事件</strong>   
  我们可以监听工具栏icon的点击事件来响应用户点击浏览器工具栏插件icon的动作；但是需要注意的是，如果插件已经声明了`popup`点击弹窗，那么这里的点击事件会无效
  ```
  chrome.action.onClicked.addListener(function(tab) {
    chrome.action.setTitle({tabId: tab.id, title: "You are on tab:" + tab.id});
  });
  ```
## 2.其他交互方式
- <strong>添加输入框关键字快捷方式</strong>       
  我们可以在浏览器输入框中使用`chrome.omnibox`设置一些快捷操作，比如用户输入特定的字符串是，使用浏览器插件特定的功能
  例如：用户输入nt时，自动激活浏览器插件,根据用户之后的输入内容新开搜索tab,实现如下：
  ```
  // manifest.json
  {
    "name": "Omnibox New Tab Search",
    ...
    "omnibox": { "keyword" : "nt" }, // 声明快捷字符
    "default_icon": {
      "16": "newtab_search16.png",
      "32": "newtab_search32.png"
    }
    ...
  }
  ```
  当用户在浏览器输入框中输入`nt`时，自动激活浏览器插件，这个时候浏览器输入框状态变化为展示插件的`灰色16icon` + `插件名` + 输入，效果如下所示：
  ![omnibox active](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/T0jCZDUVfuEANigPV6bY.png?auto=format&w=952)
  在激活之后我们还需要实现监听用户输入来自动打开搜索tab的能力，这里我们可以通过监听`chrome.omnibox.onInputEntered`事件来实现，如下：
  ```
  chrome.omnibox.onInputEntered.addListener(function(text) {
    // Encode user input for special characters , / ? : @ & = + $ #
    const newURL = 'https://www.google.com/search?q=' + encodeURIComponent(text);
    chrome.tabs.create({ url: newURL });
  });
  ```
- <strong>添加浏览器右键菜单栏选项</strong>           
  可以使用`chrome.contextMenus`api来添加浏览器右键菜单项，使用前需要在`manifest.json`中声明`contextMenus`的权限,如下所示：
  ```
  {
    "name": "Global Google Search",
    ...
    "permissions": [
      "contextMenus",
      "storage"
    ],
    "icons": {
      "16": "globalGoogle16.png",
      "48": "globalGoogle48.png",
      "128": "globalGoogle128.png"
    }
    ...
  }
  ```
  其中我们声明的16的icon则会展示在右键菜单项那一行的左侧,如下：
  ![16icon](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/jpA0DLCg2sEnwIf4FkLp.png?auto=format&w=600)
  声明权限之后我们就可以调用`chrome.contextMenus.create(config)`来创建自定义的菜单项;需要注意的是，添加菜单栏需要在浏览器插件的`runtime.onInstall`（安装的时候）的监听事件中添加
  下面展示一个具体的例子：
  ```
  // background.js
  const tldLocales = {
    'com.au': 'Australia',
    'com.br': 'Brazil',
    'ca': 'Canada',
    'cn': 'China',
    'fr': 'France',
    'it': 'Italy',
    'co.in': 'India',
    'co.jp': 'Japan',
    'com.ms': 'Mexico',
    'ru': 'Russia',
    'co.za': 'South Africa',
    'co.uk': 'United Kingdom'
  };
  chrome.runtime.onInstalled.addListener(async () => {
    for (let [tld, locale] of Object.entries(tldLocales)) {
      chrome.contextMenus.create({
        id: tld,
        title: locale,
        type: 'normal',
        contexts: ['selection'],
      });
    }
  });
  ```

- <strong>添加浏览器插件的快捷键</strong>               
  浏览器插件可以通过`Command Api`来定义一些浏览器动作的快捷键   
  需要在`manifest.json`文件中的`commands`声明快捷键的设置，如下所示。其中`commands`下的每一个key代表一个快捷键声明，快捷键的声明结构可以查看[规则](https://developer.chrome.com/docs/extensions/reference/commands/#usage)
  ```
  // manifest.json
  // 声明快捷键
  {
    "name": "Tab Flipper",
    ...
    "commands": {
      "flip-tabs-forward": {
        "suggested_key": {
          "default": "Ctrl+Shift+Right",
          "mac": "Command+Shift+Right"
        },
        "description": "Flip tabs forward"
      },
      "flip-tabs-backwards": {
        "suggested_key": {
          "default": "Ctrl+Shift+Left",
          "mac": "Command+Shift+Left"
        },
        "description": "Flip tabs backwards"
      }
    }
    ...
  }
  ```
  声明快捷键之后，需要为快捷键添加具体的事件响应，我们可以通过`background script`来添加`commands.onCommand`事件（快捷键包含特定的键盘按键触发）,如下所示：
  ```
  // background.js 
  chrome.commands.onCommand.addListener(command => {
    // command will be "flip-tabs-forward" or "flip-tabs-backwards"

    chrome.tabs.query({currentWindow: true}, tabs => {
      // Sort tabs according to their index in the window.
      tabs.sort((a, b) => a.index - b.index);
      const activeIndex = tabs.findIndex((tab) => tab.active);
      const lastTab = tabs.length - 1;
      let newIndex = -1;
      if (command === 'flip-tabs-forward') {
        newIndex = activeIndex === 0 ? lastTab : activeIndex - 1;
      } else {  // 'flip-tabs-backwards'
        newIndex = activeIndex === lastTab ? 0 : activeIndex + 1;
      }
      chrome.tabs.update(tabs[newIndex].id, {active: true, highlighted: true});
    });
  });
  ```
  由于每个浏览器插件都可以自定义右键菜单，为了便于管理，浏览器要求每个插件只能占据一个浏览器右键的菜单项位置，如果设置了多个菜单项，那么会自动集合到一个一级菜单，设置的菜单项集合在对应的二级菜单项中，效果如下所示：
  ![context menu](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/LhrliaEhN82maJmeNp7f.png?auto=format&w=1000)

- <strong>定制浏览器默认页面</strong>
  浏览器插件支持通过自己开发的页面来覆盖浏览器默认的页面，支持覆盖的页面包含新开tab页面、收藏夹页、历史记录页，分别对应的配置为`newtab`、`bookmarks`、`history`
  > 注意，一个浏览器只能覆盖上述三个页面中的一个页面；并且覆盖的页面不能使用`<script>`标签引入行内js

  对应的自定义方式如下：
  ```
  // manifest.json
  {
    "name": "Awesome Override Extension",
    ...

    "chrome_url_overrides" : {
      // 路径是相对于浏览器顶层目录
      "newtab": "override_page.html" // 自定义新开tab页
      // "bookmarks": "override_page.html" // 自定义收藏夹页
      // "history": "override_page.html" // 自定义历史记录页
    },
    ...
  }
  ```
- <strong>发送系统通知</strong>
  使用浏览器插件api`chrome.notifications.create`可以主动向用户发送系统通知，效果如图所示：  
  ![系统通知](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/e5S112AtwfnA5o64JrGg.png?auto=format&w=1000)
  但是需要注意，使用该api需要声明插件的`notifications`权限，如下：
  ```
  // manifest.json
  { 
    "name": "Drink Water Event Popup",
  ...
    "permissions": [
      "alarms",
      "notifications", // 声明 notifications系统通知权限
      "storage"
    ],
  ...
  }
  ```
