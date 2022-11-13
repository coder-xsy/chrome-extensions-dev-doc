# 浏览器插件不同模块之间信息传递
既然浏览器插件的`content script`是运行在浏览器页面的上下文中，那么他们就需要一些方式来和插件的其他组成部分来进行信息交流.例如一个RSS阅读器插件可能通过`content script`来检查当前页面的信息流内容，然后通知给`background script`在页面上展示不同的icon。   
插件与插件之间或者插件不同组成部分之间通过特定的方式进行信息传递，他们可以实现端到端的稳定信息传输，既可以发送消息，也可以使用相同的方式实现信息的接受,他们之间的信息格式以JOSN为消息格式，本次将介绍一个简单的信息方式实现单次信息传递和一种负责的方式建立长链接进行持续的消息传递,如果开发者可以知道其他浏览器插件的插件id,还可以实现跨插件的信息传递
## 单次信息传递
如果你仅仅希望发送一些简单JSON数据给浏览器插件的其他部分（也可以选择监听消息发送出去之后对方的响应），那么你可以使用`runtime.sendMessage`和`tabs.sendMessage`来实现`content script`到浏览器插件其他部分之间的消息相互传递,这两个api支持传入一个可选的额callback来处理接收消息侧的响应消息(如果接收方有响应的话)
`content script`中发送消息的例子如下所示：
```
chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
    console.log(response.farewell);
});
```
相应的，从浏览器插件发送消息给`content script`和上述例子类似，不同之处在于需要声明接收方所在的tab(`content script` 运行所在的webpage 所在的浏览器tab),如下所示：
```
// 先通过条件筛选获取当前浏览器窗口内当前打开的tab,此tab id用来标示消息接收方
chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
  chrome.tabs.sendMessage(tabs[0].id, {greeting: "hello"}, function(response) {
    console.log(response.farewell);
  });
});
```
消息接收方接收消息的方式是通过`runtime.onMessage`添加事件监听，消息接收的方式在`content script`和浏览器插件是相同的，如下所示：
```
// request 为消息内容，一般为json类型
// sender 为消息发送方的描述对象
// sendResponse 可以用来发送反馈响应消息给消息发送方
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
    console.log(sender.tab ?
                "from a content script:" + sender.tab.url :
                "from the extension");
    if (request.greeting === "hello")
      sendResponse({farewell: "goodbye"});
  }
);
```
> 需要注意：
> 1、sendResponse 为异步函数，如果希望同步调用，可以在`addListenser`的函数最后返回true
> 2、如果存在多个web page 中的`content script`同时使用`runtime.onMessage`添加了监听事件并且对于同一消息都发送了响应消息，只有第一个发送的response会成功，其他的`sendResponse`会被忽略
> 3、sendResponse只能异步调用，或者显示的在消息事件中显式`return true`来注明是同步响应，`sendMessage`的回调函数只有在没有消息接收方设置`return true`或者sendResponse被浏览器的内存管理回收的情况下才会被触发

## 持续信息传递方式
有时候在浏览器插件和`content script`之间需要持续传递数据信息，这种情况下需要创建一个持续存在的交流渠道。浏览器插件提供了`runtime.connect`和`tabs.connect`来建立一个持续存在的信息交流通道,我们还可以给每个通道赋予一个独立的名字来区分不同的消息通道

长链接的信息传递方式的一个使用场景就是具有自动表单填写功能的浏览器插件，`content script`可以创建一个长链接来和浏览器插件进行信息交互，`content script`可以将用户在web page上填写的表单内容自动发送给插件，然后插件使用这些内容来实现自动登录等功能;这个长链接支持浏览器插件和和`content script`通过持续的信息传输保持状态共享

当创建一个长链接的时候，函数会返回一个`runtime.Port`对象，我们可以直接调用这个对象实例的方法来实现通过这个长链接进行消息发送和信息接受；    
下面这个例子展示`content scipt`如何创建一个长链接并且通过它发送并接受消息
```
var port = chrome.runtime.connect({name: "knockknock"});
port.postMessage({joke: "Knock knock"});
port.onMessage.addListener(function(msg) {
  if (msg.question === "Who's there?")
    port.postMessage({answer: "Madame"});
  else if (msg.question === "Madame who?")
    port.postMessage({answer: "Madame... Bovary"});
});
```
而通过浏览器插件创建长链接给`content script`发送并接受消息和上面的方式类似，不同之处在于需要在发送消息是选择具体的浏览器tab对应的页面来作为消息接收方,在创建长链接的时候选择使用的api为[`tabs.connect`](https://developer.chrome.com/docs/extensions/reference/tabs#method-connect)
```
chrome.tabs.connect(
  tabId: number,
  connectInfo?: object,
)
```

上面介绍了创建一个长链接的方式，但是作为一个持续存在的信息传递方式，长链接类似TCP一样，也需要长链接的另外的一个端监听connect请求并且做出响应，然后才能搭建起端到端的稳定持续的信息传输通道

所以为了能够对于`connect`的请求作出响应，我们可以通过`runtime.onConnect`添加监听事件来作为`connect`的响应，这在浏览器插件或者其他`content script`中的方式是相同的，当浏览器插件的一部分（包含`content script`）发起了创建connect的请求，那么添加的响应时间会被触发，并且会将`runtime.Port`对象传递给响应函数作为入参，这样就可以和链接发起的那一端进行消息传递了，如下所示：
```
chrome.runtime.onConnect.addListener(function(port) {
  console.assert(port.name === "knockknock");
  port.onMessage.addListener(function(msg) {
    if (msg.joke === "Knock knock")
      port.postMessage({question: "Who's there?"});
    else if (msg.answer === "Madame")
      port.postMessage({question: "Madame who?"});
    else if (msg.answer === "Madame... Bovary")
      port.postMessage({question: "I don't get it."});
  });
});
```
### `runtime.Port`的生命周期
一旦通过调用`tabs.connect`、`runtime.connect`、`runtime.connectNative`这些api创建了`runtime.Port`对象，就可以使用Port对象的postMessage方法进行信息传递。
那么有一种场景，在一个tab中多帧web page都通过插件`tabs.connnect`导致触发了多次`runtime.onConnect`事件，类似，一个webpage中多次调用`runtime.connnect`触发多次浏览器插件的`onConnect`的监听事件

所以有时候你可能想知道一个长链接什么时候被关闭了，例如你可能需要同时和不同的渠道进行信息传输,为此，你可以通过监听`runtime.Port.onDisconnect`事件来指导长链接关闭的事件,这个事件会在消息传输方的另一侧没有可用的`runtime.Port`的时候触发，包括如下场景：
- 在另外一端没有`runtime.onConnect`的监听函数
- 使用`runtime.Port`的tab页面被卸载了（比如: 页面跳转）
- 通过`connect`方法创建链接的一方（插件）被卸载了
- 通过接收`onConnect`接受消息的一方全部被卸载了
- 在另外一方调用了`runtime.Port.disconnect`之后，`runtime.Port.onDisconnect`会在相反的一方触发;需要注意，如果端A同时添加了多个长链接，同时和多个消息发送方进行信息传递，然后端A调用`runtime.Port.disconnect`之后只会断开当次和她信息传递的一方的链接，也之有这一方的`runtime.Port.onDisconnect`事件会被触发，其他的消息发送方的链接不会断开

## 跨插件的信息传输
除了插件内不同组成部分之间的信息传递，浏览器还提供了跨浏览器的信息传输方式,这样你就可以暴露一些api提供给其他浏览器插件使用
和浏览器内部的信息传输方式类似，也区分简单信息传递和构建长链接实现的持续信息传递，只不过添加信息响应和长链接响应的api需要使用`runtiem.onMessageExternal`、`runtime.onConnectExternal`，下面是两种不同信息传输的例子：
```
// For simple requests:
chrome.runtime.onMessageExternal.addListener(
  function(request, sender, sendResponse) {
    if (sender.id === blocklistedExtension)
      return;  // don't allow this extension access
    else if (request.getTargetData)
      sendResponse({targetData: targetData});
    else if (request.activateLasers) {
      var success = activateLasers();
      sendResponse({activateLasers: success});
    }
  });

// For long-lived connections:
chrome.runtime.onConnectExternal.addListener(function(port) {
  port.onMessage.addListener(function(msg) {
    // See other examples for sample onMessage handlers.
  });
});
```
上面是接受消息和接受长链接建立的方式，在跨浏览器信息传递的行为中，发动消息和创建长链接的方式也和浏览器内信息传递类似，唯一的不同是你需要传入浏览器插件的id来定义信息接收方的浏览器插件,如下所示：
```
// The ID of the extension we want to talk to.
var laserExtensionId = "abcdefghijklmnoabcdefhijklmnoabc";

// Make a simple request:
chrome.runtime.sendMessage(laserExtensionId, {getTargetData: true},
  function(response) {
    if (targetInRange(response.targetData))
      chrome.runtime.sendMessage(laserExtensionId, {activateLasers: true});
  }
);

// Start a long-running conversation:
var port = chrome.runtime.connect(laserExtensionId);
port.postMessage(...);
```
## 从webpage 发送信息给浏览器插件
之前我们介绍了可以通过`content script`来实现将webpage的信息传输给浏览器插件，但是其实webpage也可以直接和浏览器进行通信，但是这里需要注意的是：<strong>只有wbepage可以主动发起和插件的信息传输，插件只能被动接受</strong>

前提：我们需要在浏览器插件的manifest.json中进行对应的声明，确认插件允许哪些webpage进行信息传输
```
"externally_connectable": {
  "matches": ["https://*.example.com/*"]
}
```
这样浏览器会暴露插件api给这些符合要求的webpage，但是需要注意，matches的正则必须包含准确的二级域名，其中*.apppot.com这个是不被允许的;
符合要求的webpage可以直接使用对应的api(eg:`runtime.sendMessage`、`runtime.connect`)来发送数据给指定的浏览器插件
```
// The ID of the extension we want to talk to.
var editorExtensionId = "abcdefghijklmnoabcdefhijklmnoabc";

// Make a simple request:
chrome.runtime.sendMessage(editorExtensionId, {openUrlInEditor: url},
  function(response) {
    if (!response.success)
      handleError(url);
  });
```
而在浏览器侧和浏览器之间的信息传递方式一样，可以使用`runtime.onMessagexternal`、`runtime.onConnectExternal`来接受webpage发送过来的数据传输请求和创建长链接请求（所以可以将和webpage的直接通信看作一种跨浏览器插件的信息传递的一种特殊类型）,如下:
```
chrome.runtime.onMessageExternal.addListener(
  function(request, sender, sendResponse) {
    if (sender.url === blocklistedWebsite)
      return;  // don't allow this web page access
    if (request.openUrlInEditor)
      openUrl(request.openUrlInEditor);
  });
```

## 和原生应用进行信息传递
插件也可以和那些作为原生信息接受端口的原生应用进行消息传递,具体的方式可以参考：[`native message`](https://developer.chrome.com/docs/apps/nativeMessaging/)

## 信息传递的安全考虑

### content script可信任度不高
content script比浏览器插件的background page的可信度低，比如一个恶意的webpage可能通过当前页面的content script阻碍浏览器的渲染进程，需要考虑通过content script发送过来的数据可能被攻击者拦截或篡改过，所以需要确保接受到的信息进行危害处理过并且判断可信度;也需要考虑任何发送给content script的数据也可能危害到web page；所以我们需要对于content script发送过来的信息和请求作出限制，缩小可能的影响范围

### 跨站点的脚本攻击
当你收到来自其他浏览器插件或者`content script`发来的数据时，你应该需要足够慎重处理这些数据避免收到脚本攻击;这些攻击脚本可能用来在浏览器插件的background page中执行或者是在webpage 的`content script`中执行,需要额外注意来自消息数据中的脚本攻击，我们应该避免如下危险的使用方式：
```
chrome.tabs.sendMessage(tab.id, {greeting: "hello"}, function(response) {
  // 危险，可能执行一段攻击脚本
  var resp = eval("(" + response.farewell + ")");
});
```
```
chrome.tabs.sendMessage(tab.id, {greeting: "hello"}, function(response) {
  // 危险，可能引入一个恶意脚本文件
  document.getElementById("resp").innerHTML = response.farewell;
});
```
更建议如下的处理方式：
```
chrome.tabs.sendMessage(tab.id, {greeting: "hello"}, function(response) {
  // JSON.parse不会执行脚本
  var resp = JSON.parse(response.farewell);
});
```
```
chrome.tabs.sendMessage(tab.id, {greeting: "hello"}, function(response) {
  // innerText不会让攻击脚本引入到我们的允许环境中
  document.getElementById("resp").innerText = response.farewell;
});
```

## samples
可以找到一些数据传输的[具体例子](https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/mv2-archive/api/messaging)帮助你理解