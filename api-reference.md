# api 总结
chrome 浏览器为插件开发提供了很多api支持，对应不同的应用场景，类似`chrome.runtime`和`chrome.alarms`
## 前置约定
下文中，除非特殊声明，所有的`chrome.*`对应的api都是异步的，他们会直接返回一个`promise`,如果你希望监听api换回的结果，可以传递一个回调函数来获取
## Stabel APIs
|       API Name        |               说明               |
| :-------------------: | :------------------------------: |
| accessibilityFeatures | 使用`chrome.accessibilityFeature`来管理浏览器的功能范围，为了能够获取这些功范围，需要为浏览器插件声明`accessibilityFeatures.read`权限，如果还需要修改这些能力状态，还需要额外声明`accessibilityFeatures.modify`权限，需要注意的是这两个权限是独立的，不存在覆盖或者大于的能力 |
|action|使用`chrome.action` api可以用来控制浏览器插件在浏览器工具栏上icon的表现|
|alarms|`chrome.alarms`用来规划将来某个时机或者循环执行的代码逻辑|
|audio| `chrome.audio`实现对于系统安装的音频播放设备的信息，并控制播放设备。这个api现在只有在chromeOS上的kiosk模式可以使用|
|bookmarks|`chrome.bookmarks`用来控制浏览器的收藏夹|
|browserAction|使用`browserAction`用来在浏览器输入框右侧添加icon，可以为icon添加起泡提示，悬浮提示框和弹出窗|
|browsingData|`chrome.browsingData`用来移除用户的浏览数据，包含cookie，下载文件之类|
|certificateProvider|`chrome.certificateProvider`可以用来到处一份证书文件，可以用来做TLS身份验证|
|commands|`chrome.commands`用来为浏览器插件添加快捷键|
|contentSettings|`chrome.contentSettings`用来修改控制浏览器打开的网页能够使用的一些能力或数据，通常用来控制对于浏览器对于特定网页的特定表现|
|contextMenus|`chrome.contextMenus`用来添加浏览器的右键菜单中添加选项，你可以选择为特定右键选中的元素添加，例如选中的图片或者超链接|
|cookies|使用`chrome.cookies`来查询或者修改浏览器存储的cookie,同时也可以监听cookie的修改|
|debugger|`chrome.debugger`创建一个chrome调试服务，通过该api可以用来连接到一个或多个浏览器tab的调试窗口中的netword、debug Javascript、调试DOM和CSS等，可以通过`tabId`参数来连接到特定浏览器窗口|
|declarativeContent|使用`chrome.declarativeContent`可以用来根据当前浏览器窗口打开的页面内容（包括静态资源、链接、DOM...）来触发特定的插件动作，并且不需要额外授权|
|declarativeNetRequest|`chrome.declarativeNetRequest`用来拦截或者修改符合设定规则的网络请求，它允许插件不用阻塞或者获取请求内容就能够实现修改网络请求的能力，从而实现更高的安全性|
|desktopCapture|`chrome.desktopCapture`用来截屏，可以window和其他tab使用|
|devtools.inspectedWindow|`chrome.devtools.inspectedWindow`用来和当前打开调试工具的页面进行交互：获取当前窗口的`tabId`、在当前窗口执行特定的代码、重新加载当前页面或者获取当前页面请求的资源文件|
|devTools.network|`chrome.devtools.network`用来获取当前页面在开发者调试工具中展示的network信息|
|devtools.panels|`chrome.devtools.panels`用来在开发者调试页面中插入浏览器定义的调试面板|
|devtools.recorder|`chrome.devtools.recorder`用来自定义调试工具中的recorder面板|
|documentScan|使用`chrome.doucmentScan`可以获取通过系统安装的内容扫描机器扫描生成的图片内容，该api只能在chromeOs 使用|
|dom|使用`chrome.dom`可以用来调用一个DOM相关的api`openOrClosedShadowRoot`: `chrome.dom.openOrClosedShadowRoot`|
|downloads|`chrome.downloads`用来监听、操控、搜索当前浏览器的下载内容|
|enterprise.deviceAttributes|`chrome.enterprise.deviceAttributes`用来获取当前设备信息，这个api只有在插件是被组织强制安装的时候可以使用|
|enterprise.hardwarePlatform|`chrome.enterprise.hardwarePlatform`用来获取当前浏览器运行的设备的生产商和型号,这个api只有在插件是被组织强制安装的时候可以使用|
|enterprise.networkingAttributes|`chrome.enterprise.networkingAttributes`用来获取当前使用的网络的相关信息，这个api只有在插件是被组织强制安装的时候可以使用|
|enterprise.platformKeys|`chrome.enterprise.platformKeys`用来生产证书使用的key，并生成安装证书，这些证书可以用来TLS认证和网络验证|
|events|`chrome.events`下包含了所有的公共的事件名称，这些事件可以被api触发，可以通过events的事件名称的来判断什么事件被触发了|
|extension|`chrome.extesion`实现了一个公共的事件通道，可以用来在插件不同页面、插件的contentScript、不同的浏览器插件之间传递信息|
|extensionTypes|`chrome.extensionTypes`包含了浏览器插件中的一些类型定义|
|fileBrowserHandler|`chrome.fileBrowserHanlder`用来拓展浏览器自定的文件上传动作，例如拓展上传动作来实现上传文件至你指定的服务器|
|fileSystemProvider|`chrome.fileSystemProvider`用来创建一个文件系统，可以被浏览器的文件管理获取|
|fontSetting|`chrome.fontSetting`用来管理浏览器的字体设置|
|gcm|`chrome.gcm`允许app和浏览器插件通过FCM传递信息|
|history|`chrome.history`用来和浏览器的浏览历史进行交互，可以用来添加或者移除或者搜索浏览器历史浏览记录，可以用来自定义浏览器的历史记录页面|
|i18n|`chrome.i18n`用来配置插件的国际化设置|
|identity|使用`chrome.identity`来获取`OAuth2 access tokens`|
|idle|`chrome.idle`用来监听idle的状态修改|
|input.ime|使用`chrome.input.ime`可以设置自定义的IME为浏览器|
|instanceID|`chrome.instanceID`用来获取浏览器连接的对应的服务|
|loginState| `chrome.loginState`用来获取当前浏览器的登陆状态或者用来监听用户登录|
|management|`chrome.management`用来管理当前浏览器中安装或者正在运行中的插件，可以用来多个浏览器插件定义的new Tab page时的表现|
|notifications|`chrome.notifications`可以用来发送桌面通知给用户,其中可以通过消息格式来生成消息内容|
|omnibox|`chrome.omnibox`用来定义一个输入框使用的关键字，当用户在输入框输入符合关键字规则的内容时和浏览器插件进行交互|
|pageAction|`chrome.pageAction`用来在当前浏览器工具栏的输入框右侧添加icon,用来给特定的页面添加该icon,如果当前页面不可见时，该icon会置灰|
|pageCapture|`chrome.pageCapture`用来将当前窗口内容保存为MHTML|
|permissions|`chrome.permissions`用来在插件运行时声明浏览器权限，所以我们在manifest.json中之需要声明必要的权限（在安装时授权），其他权限可以在运行时声明|
|platformKeys|`chrome.platformKeys`用来运行插件使用系统安装的证书|
|power|`chrome.power`用来设置系统的电量管理设置|
|printerProvider|`chrome.printerProvider`用来暴露一些事件用来给系统的打印机管理系统,可以用来获取打印机系统的打印快照或者提交打印任务|
|printing|通过`chrome.printing`可以发送打印任务给chromebook|
|printingMetrics|使用`chrome.printingMetrics`可以获取打印参数|
|privacy|`chrome.privacy`可以控制浏览器关于用户隐私的相关设置|
|proxy|通过`chrome.proxy`可以用来管理浏览器代理设置|
|runtime|通过`chrome.runtime`用户可以获取background page上下文、获取manifest.json具体内容、监听或者响应浏览器或者插件的具体生命周期、也可以用来将相对链接转换成绝对链接|
|scripting|通过`chrome.scripting`可以在不同上下文中执行特定的script代码|
|search|`chrome.search`用来实现在给定的范围内进行查找，类似ctrl+f|
|sessions|`chrome.sessions`用来查找或者重新恢复浏览器本次会话中打开的tab或者window|
|storage|`chrome.storage`用来获取、存储、监听浏览器的本地缓存信息|
|system.cpu|`chrome.system.cpu`用来获取系统使用的CPU基础信息|
|system.display|`chrome.system.display`用来获取系统使用的渲染内容的基础信息|
|system.memory|`chrome.system.memory`用来获取系统使用的内存信息|
|system.storage|`chrome.system.storage`用来获取系统当前的存储设备，并且在存储设备（可移动存储设备）被移除时收到监听消息|
|tabCapture|`chrome.tabCapture`可以和浏览器tab的媒体设备（设备尺寸、音频...）变化进行交互|
|tabGroups|`chrome.tabGroups`用来控制浏览器tab管理器进行交互，可以用来重新排列或者分组/展开tab group|
|tabs|`chrome.tabs`用来控制浏览器tab管理器进行交互，可以用来重新排列或者新开、修改tab|
|topSites|`chrome.topSites`用访问浏览器当前展示的窗口，用来获取最常访问的链接接|
|tts|`chrome.tts`用来将文字转化成语音播放,和`chrome.ttsEngine`指定的语音引擎结合使用|
|ttEngine|`chrome.ttEngine`用来使用语音播放引擎，如果插件声明使用了该api,则插件可以接受其他插件或者浏览器播放语音的事件和参数，也可以用来播放语音|
|types|`chrome.types`包含了浏览器预设的一些类型常量|
|vpnProvider|`chrome.vpnProvider`用来设置浏览器的VPN设置，可以用来设置使用的VPN|
|wallpaper|`chrome.wallpaper`可以设置浏览器的壁纸|
|webNavigation|`chrome.webNavigation`可以监听导航栏的地址请求状态|
|webRequest|`chrome.webRequest`可以用来监听和分析浏览器发起的网络请求，可以用来监听、修改、阻塞网络请求|
|windows|`chrome.windows`用来管理浏览器打开的窗口， window>tab>site|

## 系统层api  
[系统层级api](https://developer.chrome.com/docs/extensions/reference/#platform_apps_apis)提供了操作浏览器之外设备的能力，比如蓝牙、粘贴板、usb、系统网络、系统其他端口
> 需要注意的是在chrome浏览器的未来规划当中，这些api将会被移除，所以这些api当前属于deprecated状态，但是目前(2022)可以用







