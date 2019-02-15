# localStorage vs cookie 的差异

## 性能
两者并无决定性差异。

## 接口
原生的接口，localStorage好用些。不过第三方库可以消除这些差异。

## localStorage
任何在这个页面运行的JS都可以访问 localStorage。 但是不同协议(http/https)下的js，不能访问各自的localStorage。

## cookie
最严格的可以设置为JS毫无访问权限，仅仅作为服务器存储个别敏感信息的手段(以加密方式存储)。  
即对于敏感数据，应该由后台控制，后台创建加密的 http only cookie. SameSite=strict (prevent CSRF). secure=true (https only).  
其次可以设置只有来源于同一个domain下的JS才能访问cookie。

## 结论
最严格的安全，按照JS不访问cookie来做。  
其次，如果要用JWT，把token之类的信息存放到cookie，仅限信任的JS访问。但如果SSO，那么要注意谁设置的cookie，这个cookie又允许谁访问。  
