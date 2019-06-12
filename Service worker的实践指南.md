# 背景
本文记录在一个Electron运行环境内添加service worker的过程。尽量让这些过程可以无障碍的在浏览器里实施。为下一个项目采用service worker留下一份行动指南。

# 采用service worker的目的
1. 离线查看。离线查看可以大大优化断网的体验，给用户明确和友好的提示。
2. 性能。某些资源不需要重复从网络获取。

# 采用service worker的困难
1. 如果sw代码有bug，很痛苦。可能要等浏览器主动更新sw文件。这时需要sw的总开关。
2. sw相当于一个中间层，要考虑非实时数据和资源带来的影响。
3. sw本身的bug或feature。比如 https://developers.google.com/web/updates/2018/06/fresher-sw 这个来解决sw文件每次都更新的场景。

# 技术依赖@vue/cli@3.7(workbox@3.6.2)
workbox最新版是4.x，相比3.x有breaking change，不过在本文中，没有根本性的影响。

# 使用@vue/cli生成项目后，根据文档指引来配置pwa
https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa

## 采用 generatesw 还是 injectmanifest ？
采用 generatesw 的场景：(from https://developers.google.com/web/tools/workbox/modules/workbox-build)
* When to use generateSW
1. You want to precache files.
1. You have simple runtime configuration needs (e.g. the configuration allows you to define routes and strategies).
* When NOT to use generateSW
1. You want to use other Service Worker features (i.e. Web Push).
1. You want to import additional scripts or add additional logic.

采用 injectmanifest 的场景：需要更复杂的控制

## skipWaiting ？
只在登录页才触发skipWaiting。因为某些页面是不能刷新的。  
1. 在app scope通过workbox监听waiting事件。得到waiting事件即表明新的sw已经准备好了。
1. 在登录页去触发skipWaiting事件。
可以在window上标记是否有新的sw，把workbox附加到window上。  

但因为我的应用是SPA，并且有登录页，可以采用简单方式来做。
1. 因为只有在登录页有刷新的功能，所以可以在登录时检测app.[md5].js是否存在，如果不存在则刷新页面，要刷两次。
1. workbox配置 skipWaiting: true, clientsClaim: true,

## offline and app shell ?
因为我的应用必须通过网络来获取应用初始设置，即没有离线查看的需求和可能性，所以没有做app shell。  
同时，因为网络是必须的，所以肯定可以获取index.html/js/css。

## 测试
@vue/cli不建议在测试环境下开启service worker，建议我们先做build之后再启用静态资源服务。  
我尝试用serve(npm包)来做转发，但它的proxy很不好用。比如我想对符合 `/api/**` 路径的请求 转发到 `https://xxx:port/api/**` 看了文档，试了好久都没有成功。  
后来通过http-server(npm包)很简单的就完成了。  

## service worker的开关
因为service worker的资源被缓存，并且网络请求在本地被拦截处理，所以最糟的情况是发布了有bug的代码，但无法更新代码。  
又或者是service worker相关的代码有bug，想暂停service worker。  
注意：浏览器会在24小时后更新service worker代码，避免永远在旧代码中循环。  
所以原则是永远有个请求能脱离浏览器的控制发向server。  
对于我的应用来说，旧的js资源代码会在发布后被删除，所以我在登录时在这个资源路径上加上时间戳，通过head请求判断这个资源是否存在。  
如果不存在，则有新版本发布，需要刷新页面获取新版本，记得要刷两次。  
第一次刷新时通过localStorage添加标识，再次进入页面时查看此标识，做第二次刷新。  
