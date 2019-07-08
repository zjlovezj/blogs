最近在项目中有个需求有非常多的前后端消息互换，经过一些讨论和调研，决定使用websocket来实现需求。  
websocket不是很新的技术，网络上的资料非常多，我这里记录一下我认为重要的点以及我归纳的点。  

## 什么情况下用websocket？
需要后端主动推送消息给前端的情况下，用websocket是最好的。  
替代方案：
1. 轮询。性能差。
2. 用户触发。体验差。

## websocket能支持多少连接？
根据网上查询，单机(非顶配)可以支持几百万的连接。

## websocket也有 load balance 的方案
最简单的就是根据ip来分流。

## websocket消息可靠吗？
尽管websocket是基于tcp的，但由于网络复杂的原因，websocket的消息可能不能成功送达，比如局域网的NAT就可能超时断掉。50秒发一次心跳，避免半死不活。  
这个时候，要有心跳，保证连接不断掉，比如浏览器发ping，等着接收服务器的pong.  
然而还是可能断掉，或消息没有被接收，所以要能重连，重要消息必须等回执才能说发送成功。  

## websoket的状态机(state machine)
open(failed) -> error -> close(CloseEvent.code) 在这里去判断是否重连 (error总是导致close)  
open(sucess) -> send/message -> close  
open(sucess) -> send(failed? 应该仅在正常情况下发送)  
client/server都可以调用close事件，onclose都会执行，通过code来判断(4000以上)  

## websocket发送失败？
websocket消息发送为确保成功，应该通过onmessage来保证服务器接收到消息。

## websocket超时？
网络节点是可以设置超时的。send时超时是以onerror事件通知吗？

## websocket重连？
重连需要关闭吗？看起来不需要了，只是沿着创建的流程重新走一遍就可以了。

## websocket认证？
websocket不能设置header。但websocket在连接的第一步发出的是http请求，这里是有cookie的，所以可以根据cookie来认证，认证通过则connection建立。

## websocket的优势和代价？
websocket就是长连接。  
所谓长链接就是双方有记忆。  
记忆的代价：  
1. 需要存储。
2. 需要占用资源，内存、句柄、端口等。
3. 需要相对复杂的控制协议来开启和关闭。

## websocket不适合的地方？？
1. 业务模式需要request/response的请求和应答。http是比较合适的。websocket发送后不管对方的回复。
