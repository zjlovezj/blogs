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
