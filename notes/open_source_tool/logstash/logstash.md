## logstash处理apache日志常见问题汇总

> ###1. 过滤过期的日志
> <pre><code>
> filter {
>     date {
>         match => ["datetime" , "UNIX"]
>     }
>     ruby {
>         code => "event.cancel if 5 * 24 * 3600 < (event['@timestamp']-::Time.now).abs"
>     }
> }
> </pre>




> ###2. apache日志中日志级别有很多缩写方式，且大小写也没有定规。为方便统一检索，将所有级别统一成全词大写
> 
> <pre><code>
> filter {
>     ruby {
>         code => "
>             obj = 'INFO WARNING ERROR ALERT TRACE DEBUG NOTICE CRITICAL FATAL SEVERE EMERGENCY'
>             event['level'] = obj.scan(/\b#{event['level'].to_s}[a-zA-Z]*/i).at(0)
>         "
>     }
> }
> </pre>


> ###3. logstash处理多行，可以在input时处理，也可以在filter处理
> 在input时处理的配置：
> <pre><code>
> input {
>     stdin {
>         codec => multiline {
>             pattern => "^\["
>             negate => true
>             what => "previous"
>         }
>     }
> }
> </pre>
> 
> 在filter中处理的配置：
> <pre><code>
> filter {
>     multiline {
>         patterns_dir => "/Users/lanyonm/logstash/patterns"
>         pattern => "^\["
>         negate => true
>         what => "previous"
>     }
> }
> </pre>
> 
> 其中negate的值可以是true或者false，默认值是false。what的值可以是"previous"或"next"。
> negate和what的值组合在一起，对应multiline的不同处理逻辑。
> |  negate  |  what  |  处理逻辑  |
> | :------: | :------: | :------ |
> |  true  |  previous  |  所有不匹配pattern的行与前一行合并，直到有匹配pattern的行出现  |
> |  true  |  next  |  所有不匹配pattern的行与之后出现的行合并，直到有匹配pattern的行出现  |
> |  false  |  previous  |  所有匹配pattern的行与前一行合并，直到有不匹配pattern的行出现  |
> |  false  |  next  |  |  所有匹配pattern的行与之后出现的行合并，直到有不匹配pattern的行出现  |
> 
> 两个比较容易理解的例子：
> * Java日志中以空格或tab开头的行，和前面没有空白字符开头的行合并。
> <pre><code>
> 2014-01-09 17:32:25,527 -0800 | ERROR | com.example.controller.ApiController - Request exception javax.xml.ws.WebServiceException: Failed to access the WSDL at: https://api.example.com/DataServices/Data?WSDL. It failed with:
>     Connection reset.
>     at com.example.webservices.Data.<init>(Data.java:50)
>     at com.example.service.soap.DataService.submitRequest(DataService.groovy:28)
>     at com.example.service.request.RequestService.addRequest(RequestService.groovy:26)
>     at com.example.controller.ApiController.request(ApiController.groovy:692)
>     at grails.plugin.cache.web.filter.PageFragmentCachingFilter.doFilter(PageFragmentCachingFilter.java:200)
>     at grails.plugin.cache.web.filter.AbstractFilter.doFilter(AbstractFilter.java:63)
>     at org.apache.jk.server.JkCoyoteHandler.invoke(JkCoyoteHandler.java:190)
>     at org.apache.jk.common.HandlerRequest.invoke(HandlerRequest.java:311)
>     at org.apache.jk.common.ChannelSocket.invoke(ChannelSocket.java:776)
>     at org.apache.jk.common.ChannelSocket.processConnection(ChannelSocket.java:705)
>     at org.apache.jk.common.ChannelSocket$SocketConnection.runIt(ChannelSocket.java:898)
> </pre>
> 
> 可如下配置:
> <pre><code>
> filter {
>     multiline {
>         type => "somefiletype"
>         pattern => "^\s"
>         what => "previous"
>     }
> }
> </pre>
> 
> * C语言中以\结尾的行表示一行未结束，还要和后面的行连接起来
> 
> <pre><code>
> #define LOG_TO_STR(str, fmt, arg...) \
>     do { \
>         sprintf(str, "%s: " fmt, "m_prog", ##arg); \
>     } while (0)
> </pre>
> 
> 
> 脚本配置如下：
<pre><code>
> filter {
>     multiline {
>         type => "somefiletype "
>         pattern => "\\$"
>         negate => true
>         what => "next"
>     }
> }
> </pre>
