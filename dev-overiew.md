# 插件开发能力概览
本文会对浏览器插件的内容和能力做一个总结归类，方便开发者进行探索和拓展浏览器插件的功能
## 
插件的用户交互能力
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Action|控制浏览器插件在浏览器工具栏的表现和交互行为|
|Commands|给浏览器添加快捷键|
|ContextMenus|给浏览器右键添加菜单项|
|Omnibox|在浏览器地址栏定义一些关键字，功能类似快捷键|
|Overide pages|插件可以自定义浏览器的新开tab页面、书签收藏夹页、历史记录页|
|Page Actions|动态控制浏览器工具栏（输入框右侧）展示icon,比如翻译之类的 |

## 通用基础能力
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Accessibility|无障碍能力支持|
|Service Workers|监听和响应特定的事件|
|Internationalization|国际化相关|
|Identity|获取身份认证token|
|Management|控制已经安装和已经运行的浏览器插件|
|Message Passing|浏览器插件的content Script和插件之间的信息传递方式，同时也可以用来不用插件/附加设备之间的信息传递|
|Options Pages|为浏览器自定义一个页面，这个页面可以通过右键浏览器icon打开|
|Permissions|运行时修改浏览器插件的授权|
|Storage|存取浏览器插件的缓存数据|

## 修改或者监听浏览器行为
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Bookmarks|管理浏览器的收藏夹，可以新建收藏或者管理已有的收藏内容|
|Browsing Data|修改浏览器记录的用户行为|
|Downloads|管理浏览器的下载行为，可以创建、监听、搜索、修改已有的下载队列内容|
|Font Settings|修改浏览器的字体配置|
|History|处理浏览器的历史浏览记录|
|Privacy|设置浏览器的隐私政策设置|
|Proxy|处理浏览器的代理设置|
|Sessions|查找或者恢复本次浏览器会话周期内打开的tab或者window|
|Tabs|管理浏览器的tab,可以新建、修改、重新排序|
|Top Sites|获取浏览器最常访问的站点|
|Themes|修改浏览器主题|
|Windows|管理浏览器的窗口<br><small>(ps:浏览器会话周期内可以打开多个窗口，每个窗口可以有多个tab)</small>|

## 处理浏览器打开的站点
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Active Tab|配置插件生效的url规则|
|Content Settings|对于当前页面的运行环境进行设置，可以控制cookie、javascript、plugin等|
|Content Scripts|在当前页面的上下文中执行特定的js代码|
|Cookies|对当前页面的cookie管理进行监听和操作|
|Cross-Origin XHR|请求远程服务器的数据|
|Declarative Content|可以用来根据当前浏览器窗口打开的页面内容（包括静态资源、链接、DOM...）来触发特定的插件动作，并且不需要额外授权|
|Desktop Capture|获取当前屏幕的截屏,不区分window和tabs|
|Page Capture|存储当前页面的资源保存为MHTML|
|Tab Capture|和当前tab页面中的媒体内容进行交互|
|Web Navigation|监听浏览器路由状态变更|
|Declarative Net Request|声明或动态定义一些规则来修改或拦截当前页面的网络请求|

## 打包发布相关
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Chrome Web Store|通过本地网络或者chrome插件平台更新插件|
|Other Deployment Options|特定局域网部署浏览器插件|

## 拓展和调试
| api名称/能力名称 | 说明  |
| :--------------: | :---: |
|Debugger|拓展浏览器默认的调试能力（network debugger DOM CSS）|
|Devtools|为浏览器新增调试面板|








