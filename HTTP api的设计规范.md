# 前言
HTTP api是最通用的接口。http协议具有通用性，并且web的架构已经通过互联网得到过验证。但是设计api接口并没有强制规定，所以我们必须学习规范才能保证良好的设计。

## 基石
Roy Fielding的关于设计web services的论文，首次提出了ReST。  
[https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

## 通用原则

1. URL来表示resource
1. resource可以通过URI来定位
1. URL由resource的名词组成，不应该包含动作/动词
1. 应该使用复数来表示resource
1. 使用GET/POST/PUT/DELETE来操作resource

## URL
1. 可以嵌套resource
1. 不要包含version，不利于升级。可以在header中包含version。
1. 分页: page/size/limit/offset //TODO
1. sort: sort=-created_at。通过"-"来表示ASC或DESC
1. 多resource表示: /payments/1234,444,555,666

## HEADER

## RESPONSE

## 错误处理
采用http的status code来表示处理结果。用户只用熟悉一套状态码，而不是每一个产品的状态码。  
每一个接口的返回状态都需要单独处理，而不是将返回数据传给业务逻辑。 

## 健康检查
检查一个URL，看返回值是否正确，比如 /health_check

## 思考
### 常用非REST方案
{code, data, message}  
code=0代表正常，非0可以来表示错误代码  
data表示返回的数据  
message来表示错误信息  

### 著名非REST接口
aws的ec2即是，以及模仿它的阿里云api。  
例子： https://ec2.amazonaws.com/?Action=AllocateAddress&AUTHPARAMS  
它的这个api并非它成功的原因，不要模仿它。而且它们的SDK缓解了http api的不规范设计。

## 参考信息
1. [https://github.com/gocardless/http-api-design](https://github.com/gocardless/http-api-design)
1. [https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)
1. [https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f](https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f) 特别是status code的部分

