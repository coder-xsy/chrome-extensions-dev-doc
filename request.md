# chrome.declarativeNetRequest
####  描述  
<code>chrome.declarativeNetRequest</code>这个API可以通过设置规则来拦截或者修改浏览器网络请求，不用通过说动拦截或者查看具体的网络请求来实现，拥有更好的隐私保密性
####  需要的授权   
`declarativeNetRequest`   
`declarativeNetRequestWithHostAccess`   
`declarativeNetRequestFeedback`   
<code>[host permissions](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/)</code>    
还有一些[其他权限](https://developer.chrome.com/docs/extensions/mv3/permission_warnings/#permissions_with_warnings)可选   
####  浏览器版本要求
<span>chrome 84+</span>

## Manifest配置
插件必须要声明`declarativeNetRquest`或者`declarativeNetRequestWidthHostAcess`(chrome96+)权限才能使用这个`chrome.declarativeNetRequest`下的api，同时如果希望能够拦截、重定向或者修改网络请求，还必须声明host permissions权限，`declarativeNetRequestWithHostAccess`这个权限用来设置插件希望能够处理的URL,`declarativeNetRequestFeedback`如果需要使用包含具体数据的网络请求相关的函数和事件需要申明   
同时为了能够设置静态规则（也支持通过api动态设置规则)，还需要在manifest中配置`declarative_net_request`，这个配置的值是一个只包含key(`rule_resources`)的对象，其中rule_resources的值是[规则]()数组  
<strong>其中申明的静态网络规则的数量是有限的，查看具体的[限制](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#rule-resources)</strong>

```
{
  "name": "My extension",
  ...

  "declarative_net_request" : {
    "rule_resources" : [{
      "id": "ruleset_1",
      "enabled": true,
      "path": "rules_1.json"
    }, {
      "id": "ruleset_2",
      "enabled": false,
      "path": "rules_2.json"
    }]
  },
  "permissions": [
    "declarativeNetRequest",
    "declarativeNetRequestFeedback",
    "*://example.com/*"
  ],
  ...
}
```
## netRequest规则
规则大致可以包含几种动作类型
- 拦截/阻塞网络请求
- 取消拦截到的网络请求
- 重定向网络请求
- 修改请求的header    
  
一个规则的数据类型，可以查看[demo](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#rule-resources)
```
{
  id: number, // 规则id
  priority: number // 权重，用来区分rule的优先级
  action: {
    "type": string // 代表规则对于network的操作类型，属于可枚举
  }
  condition: {
    urlFilter: string // 字符串或正则字符串 过滤url
    domains: string[] // 规则生效的域名集合
    resourceTypes: string[] //拦截的资源请求类型,例如： ["srcipt"]
  }
}
```

## 动态规则和浏览器会话周期内有效规则
插件可以使用api`updateDynamicRules`和`updateSessionRules`来动态添加或者移除网络规则，两者的差别是规则存在的有效期不一样
- 通过`updateDynamicRules`添加的规则在打开浏览器的会话期内和插件更新的时候有效
- 通过`updateSessionRules`添加的规则不会跨浏览器sessions生效，并且规则是保留在浏览器的运行时内存内的
- 同时通过这两个API添加的rules是有数量上限的，默认是5000,由常`MAX_NUMBER_OF_DYNAMIC_AND_SESSION_RULES`控制    
  
##  动态更新生效的静态规则
插件能够通过api`updateEnabledRulesets`动态更新静态规则(静态规则是指通过rule_resources在manifest中声明的规则，动态规则可以通过api`updateDynamicRules`和`updateSeesionRules`来更新)   
- 一次打开最大的规则集合数量数量上限由常量`MAX_NUMBER_OF_ENABLED_STATIC_RULESETS`控制，默认是10
- 浏览器对于所有插件的规则数量是有一个整体的数量上限`30000`，可以通过api`getAvailableStaticRuleCount`来检查当前能够打开的最大数量
- 静态规则集合能够跨sessions生效，但是不能跨浏览器插件版本生效，每次插件更新或者第一次安装的时候浏览器会自动检查rule_resources的值，只有最新的配置才会生效

## rule的优先级
在一个请求发起之前，每个插件都会有一个时间可以消费自己定义的规则动作类型(`action.type`),例如：
- action `block` 能够拦截请求呀
- action `redirect`和`upgradeSchema`能够重定向请求
- action `allow`和`allowAllRequests`能够捕捉到所有请求
  
<strong>执行顺序:</strong>  
如果一个action类型超过一个插件定义了rule,那么浏览器按照rule定义的权重`priority`来确认执行的先后顺序，如果权重相同，那么最后安装的插件的规则优先执行   
如果在一个插件内的多个action,优先按照权重`priority`来排列窒息顺序，如果拥有相同的执行顺序，那么按照action类型来执行，action的优先级为`allow`>`allAllRequests`>`block`>`upgradeSchema`>`redirect`    
如果一个请求没有被block或者redirected，那么`modifyHeader`的action规则会按照`插件安装顺序（近优先级高）`>插件内的`权重`的顺序执行；在插件内，如果一个`modifyHeaders`的规则权重比`allow`和`allowAllRequests`规则权重底，则不会被执行    
如果存在多个 `modifyHeaders`规则修改同一个header的情况，会按照他们的权重顺序按照如下规则进行处理  
- rule1针对一个header key 有`append`操作，那么优先级比该rule1权重低的的规则中的`set`和`remove`不会被运行
- rule1针对一个header key 有`set`操作，那么比他优先级低的modify操作都不被运行，除非是`append`操作
- rule1针对一个header key 有 `reomve`操作，那么比他优先级低的modify操作不会被执行
  
## 具体的demo
manifest.json
```
{
  "name": "declarativeNetRequest extension",
  "version": "1",
  "declarative_net_request": {
    "rule_resources": [{
      "id": "ruleset_1",
      "enabled": true,
      "path": "rules.json"
    }]
  },
  "permissions": [
    "*://*.google.com/*",
    "*://*.abcd.com/*",
    "*://*.example.com/*",
    "https://*.xyz.com/*",
    "*://*.headers.com/*",
    "declarativeNetRequest"
  ],
  "manifest_version": 2
}
```

rules.json
```
[
  {
    "id": 1,
    "priority": 1,
    "action": { "type": "block" },
    "condition": {"urlFilter": "google.com", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 2,
    "priority": 1,
    "action": { "type": "allow" },
    "condition": { "urlFilter": "google.com/123", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 3,
    "priority": 2,
    "action": { "type": "block" },
    "condition": { "urlFilter": "google.com/12345", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 4,
    "priority": 1,
    "action": { "type": "redirect", "redirect": { "url": "https://example.com" } },
    "condition": { "urlFilter": "google.com", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 5,
    "priority": 1,
    "action": { "type": "redirect", "redirect": { "extensionPath": "/a.jpg" } },
    "condition": { "urlFilter": "abcd.com", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 6,
    "priority": 1,
    "action": {
      "type": "redirect",
      "redirect": {
        "transform": { "scheme": "https", "host": "new.example.com" }
      }
    },
    "condition": { "urlFilter": "||example.com", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 7,
    "priority": 1,
    "action": {
      "type": "redirect",
      "redirect": {
        "regexSubstitution": "https://\\1.xyz.com/"
      }
    },
    "condition": {
      "regexFilter": "^https://www\\.(abc|def)\\.xyz\\.com/",
      "resourceTypes": [
        "main_frame"
      ]
    }
  },
  {
    "id" : 8,
    "priority": 2,
    "action" : {
      "type" : "allowAllRequests"
    },
    "condition" : {
      "urlFilter" : "||b.com/path",
      "resourceTypes" : ["sub_frame"]
    }
  },
  {
    "id" : 9,
    "priority": 1,
    "action" : {
      "type" : "block"
    },
    "condition" : {
      "urlFilter" : "script.js",
      "resourceTypes" : ["script"]
    }
  },
  {
    "id": 10,
    "priority": 2,
    "action": {
      "type": "modifyHeaders",
      "responseHeaders": [
        { "header": "h1", "operation": "remove" },
        { "header": "h2", "operation": "set", "value": "v2" },
        { "header": "h3", "operation": "append", "value": "v3" }
      ]
    },
    "condition": { "urlFilter": "headers.com/123", "resourceTypes": ["main_frame"] }
  },
  {
    "id": 11,
    "priority": 1,
    "action": {
      "type": "modifyHeaders",
      "responseHeaders": [
        { "header": "h1", "operation": "set", "value": "v4" },
        { "header": "h2", "operation": "append", "value": "v5" },
        { "header": "h3", "operation": "append", "value": "v6" }
      ]
    },
    "condition": { "urlFilter": "headers.com/12345", "resourceTypes": ["main_frame"] }
  },
]
```
[执行顺序解析](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#example)
## chrome.declarativeNetRequest包含的API
所有的api同时支持promise 和callback的使用方式
### getAvailableStaticRuleCount 
返回值为 Promise(count: number)
获取当前距离最大静态规则数量上限的数量
```
chrome.declarativeNetRequest.getAvailableStaticRuleCount(
  callback?: (count) => void,
)
```   
### getDynamicRules
获取当前设置的动态规则列表
```
chrome.declarativeNetRequest.getDynamicRules(
  callback?: (rules: Rule[]) => void,
)
``` 
### getEnabledRulesets
获取当前enable的静态规则的id集合
```
chrome.declarativeNetRequest.getEnabledRulesets(
  callback?: (string[]) => void,
)
```
### getMatchedRules 
通过定义过滤规则获取设置的符合规则的rules,使用这个api需要声明`declarativeNetRequestFeedback`或者在filter中通过`tabId`设置了`activeTab`权限
```
chrome.declarativeNetRequest.getMatchedRules(
  filter?: MatchedRulesFilter,
  callback?: (details: RulesMatchedDetails) => void,
)
```
### getSessionRules   
获取session周期内的规则
```
chrome.declarativeNetRequest.getSessionRules(
  callback?: (rules:Rule[]) => void,
)
```
### isRegexSupported
根据正则过滤条件过滤出rules
```
chrome.declarativeNetRequest.isRegexSupported(
  regexOptions: RegexOptions,
  callback?: (details: RulesMatchedDetails) => void,
)
```
### updateDynamicRules
更新动态规则,`options.removeRuleIds`中的规则优先移除，然后`options.addRules`中的规则被添加
```
type UpdateRuleOptions = {
  addRules?: Rule[]
  removeRuleIds?: string[]
}
chrome.declarativeNetRequest.updateDynamicRules(
  options: UpdateRuleOptions,
  callback?: () => void,
)
```
### updateEnabledRulesets
更新静态规则,优先删除`options.disableRulesetIds`的规则，然后添加`options.enableRulesetIds`对应的静态规则
```
type UpdateRuleOptions = {
  disableRulesetIds?: string[]
  enableRulesetIds?: string[]
}
chrome.declarativeNetRequest.updateEnabledRulesets(
  options: UpdateRulesetOptions,
  callback?: () => void,
)
```
### updateSessionRules
更新当前session周期内的规则，优先删除`options.removeRuleIds`的规则，然后添加`options.addRules`对应的静态规则
```
type UpdateRuleOptions = {
  addRules?: Rule[]
  removeRuleIds?: number[] 
}
chrome.declarativeNetRequest.updateSessionRules(
  options: UpdateRuleOptions,
  callback?: () => void,
)
```

## 数据类型   
```
type MatchedRulesFilter = {
  minTimeStamp?: number // 过滤只有时间戳之后的规则
  tabId?: number // 浏览器tabid，如果设置为-1则和当前浏览器tab无关
}
type RulesMatchedDetails = RulesMatchedInfo[]
type RulesMatchedInfo = {
  tabId: number // 如果tab处于当前active，否则-1
  rule: MatchedRule
}
type MatchedRule = {
  ruleId: number // 规则id
  rulesetId: string // 规则集合id
}
type RegexOptions = RegexOptions(https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#type-RegexOptions)

```

## 查看官网的数据类型和具体的api列表
[查看](https://developer.chrome.com/docs/extensions/reference/declarativeNetRequest/#type)