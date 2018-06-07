OWIN：为 dotnet 开放的 web 服务接口
===================================

原文地址 http://owin.org/html/owin.html

版本 1.0

作者 OWIN 工作组

版权 OWIN 贡献者

开源协议 Creative Commons Attribution 3.0 Unported License

最后更新时间 20121010

目录
====

1 概述
======

本文对用于定义 OWIN，OWIN 是 .NET web 服务和 web
应用程序之间的一个标准接口。OWIN
的目标是用于服务与应用程序之间解耦【译者注：使两者间没有强关联，或者说相互不依赖】，并且成为一种开放规范，从而激励
.NET web 开发工具开源社区。

OWIN 是根据委托类型定义的，这儿没有被称作 OWIN.dll
或类似的程序集【译者注：我的理解是这用于强调 OWIN
是规范（或协议），而不是具体实现】。在宿主或应用程序中实现 OWIN
规范不会使项目引入依赖。

在本文中，C\# Action 或 Func
语法被用于一些委托类型的定义。然而，这个委托类型

可以被 F\# native functions、 CLR interfaces 或 named delegates
同等表示【译者注：由于不了解其它技术，所以使用了原文，但其表达的内容为委托类型在其它语言或技术中可以类似的定义，在此仅只是用
C\# 举例】，这是经过精心设计的。在实现 OWIN
规范时，选择一种（合适）委托表示使得其为你和你的堆栈工作。

以下关键词\"MUST\"、\"MUST NOT\"、\"REQUIRED\"、\"SHALL\"、\"SHALL
NOT\"、\"SHOULD\"、\"SHOULD
NOT\"、\"RECOMMENDED\"、\"MAY\"和\"OPTIONAL\" 按照 [[RFC
2119]{.underline}](http://www.ietf.org/rfc/rfc2119.txt)
中的描述来理解。【译者注：RFC 是计算机相关的各种说明】

2 定义
======

本文涉及到以下软件角色：

2.1 服务
--------

HTTP 服务直接与客户端连接，接着使用 OWIN
语义处理请求。服务可能需要适配层（将请求）转换为 OWIN
（识别的）语义。【译者注：后文中"服务"如果没有另外说明，都是这个意思】

2.2 Web 框架
------------

在 OWIN (管道)顶部的自包含（独立的）组件，它对外暴露对象模型或者是
API，从而使得应用程序可以使用它来便利的

处理请求，Web Framework 可能需要适配层将 OWIN
语义转换（为它识别的内容）。

2.3 Web 应用程序
----------------

一个具体的 Web 应用程序，可能构建在 Web Framework （Web
框架）的顶部，它使用 OWIN 相容的服务来运行。

2.4 中间件
----------

在服务（server）和应用程序之间构成的管道中的组件，组件出于某个目的检查、路由或者修改请求和响应报文

2.5 宿主
--------

应用程序和服务运行的进程，主要负责应用程序【OWIN
整体】的启动（和关闭）。某些服务同时也是宿主【服务同时实现了宿主的功能】。

3 处理请求
==========

一般来说，服务调用应用程序（提供环境字典参数，环境字典包含请求和响应的头和内容）；应用程序返回一个响应或指出错误。

3.1 应用程序委托
----------------

OWIN 中主要的接口称为应用程序委托或 AppFunc，应用程序委托使用
IDictionary\<string, object\> 环境（字典），并且在处理完成时返回一个
Task。

using AppFunc = Func\<

IDictionary\<string, object\>, // 环境字典

Task\>; // 方法完成时返回的任务【译者注：Task
是一种类型，后文中"任务"一般是该含义】

应用程序必须在最终完成时返回一个任务或抛出异常。

3.2 环境（字典）
----------------

环境字典存储关于请求、响应和（任何与服务状态有关）的信息。服务的职责是在请求和响应最初调用时提供它们的（报文）头集合和（报文）体流。

应用程序接下来使用响应数据填充合适的字段，然后写响应体，最后在完成时返回响应。

（1）环境字典必须非空且可更改，而且必须包含下表中列出的键列表。

（2）键的比较（相等或不等）必须使用 StringComparer.Ordinal

（3）键对应的值必须非空，除非另有说明。

除了这些增加的键，宿主、服务、中间件和应用程序等可以向环境字典中添加任意与请求或响应有关的数据。

增加键的准则和常用键列表可以在 CommonKeys addendum 文档中找到。

### 3.2.1 请求数据

  Required   Key Name                  Value Description
  ---------- ------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Yes        owin.RequestBody          A Stream with the request body, if any.Stream.Null MAY be used as a placeholder if there is no request body. See [[Request Body]{.underline}](http://owin.org/html/sec-req-body).
  Yes        owin.RequestHeaders       An IDictionary\<string, string\[\]\> of request headers. See [[Headers]{.underline}](http://owin.org/html/owin.html#33-headers).
  Yes        owin.RequestMethod        A string containing the HTTP request method of the request (e.g.,\"GET\",\"POST\").
  Yes        owin.RequestPath          A string containing the request path. The path MUST be relative to the \"root\" of the application delegate. See [[Paths]{.underline}](http://owin.org/html/owin.html#53-paths).
  Yes        owin.RequestPathBase      A string containing the portion of the request path corresponding to the \"root\" of the application delegate; see [[Paths]{.underline}](http://owin.org/html/owin.html#53-paths).
  Yes        owin.RequestProtocol      A string containing the protocol name and version (e.g.\"HTTP/1.0\"or\"HTTP/1.1\").
  Yes        owin.RequestQueryString   A string containing the query string component of the HTTP request URI, without the leading \"?\" (e.g.,\"foo=bar&amp;baz=quux\"). The value may be an empty string.
  Yes        owin.RequestScheme        A string containing the URI scheme used for the request (e.g.,\"http\",\"https\"); see [[URI Scheme]{.underline}](http://owin.org/html/owin.html#51-uri-scheme).

### 3.2.2 响应数据

  Required   Key Name                    Value Description
  ---------- --------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Yes        owin.ResponseBody           A Stream used to write out the response body, if any. See [[Response Body]{.underline}](http://owin.org/html/owin.html#35-response-body).
  Yes        owin.ResponseHeaders        An IDictionary\<string, string\[\]\> of response headers. See [[Headers]{.underline}](http://owin.org/html/owin.html#33-headers).
  No         owin.ResponseStatusCode     An optional int containing the HTTP response status code as defined in [[RFC 2616]{.underline}](http://www.ietf.org/rfc/rfc2616.txt) section 6.1.1. The default is 200.
  No         owin.ResponseReasonPhrase   An optional string containing the reason phrase associated the given status code. If none is provided then the server SHOULD provide a default as described in [[RFC 2616]{.underline}](http://www.ietf.org/rfc/rfc2616.txt) section 6.1.1
  No         owin.ResponseProtocol       An optional string containing the protocol name and version (e.g.\"HTTP/1.0\"or\"HTTP/1.1\"). If none is provided then the \"owin.RequestProtocol\"key\'s value is the default.

### 3.2.3 其它数据

  Required   Key Name             Value Description
  ---------- -------------------- ------------------------------------------------------------------------------------------------------------------------
  Yes        owin.CallCancelled   A CancellationToken indicating if the request has been canceled/aborted. See \[Request Lifetime\]\[sec-req-lifetime\].
  Yes        owin.Version         A string indicating the OWIN version. See [Versioning](http://owin.org/html/owin.html#7-versioning).
  Required   Key Name             Value Description

【译者注：以上3小节没有翻译，其它 HTTP
报文一致，如有不了解，请查阅相关内容】

3.3 报文头
----------

HTTP 请求和响应报文头都使用类型为 IDictionary\<string, string\[\]\>
的对象表示，以下要求在 RFC 2616 section 4.2 中有说明。

（1）字典必须可修改

（2）键必须是 HTTP 字段名称且没有冒号或空格，多个单词间用短横线（-）连接

（3）键的比较（相等或不等）必须使用 StringComparer.OrdinalIgnoreCase

（4）所有键值中的字符应该是 ASCII 码表中的

（5）返回的值数组假定是原始数据的拷贝，任何想要修改值数组的操作必须回溯到头字段，手动的使用
headers\[headerName\] = modifiedArray; 或 headers.Remove(header)
语法操作。

（6）键对应的值都假定是混合的格式，比如 new string\[1\] {\"value, value,
value\"}, new string\[3\] {\"value\", \"value\", \"value\"}, or new
string\[2\] {\"value, value\", \"value\"} 3种格式

（7）服务、应用程序和中间组件不应该拆分或合并非必要的报文头中的值。虽然上述的3种格式都可以互相转换，但实际上许多现有的实现只支持某种特定格式，开发人员应当通过选定或假定某种格式来灵活的支持现有的实现格式。

3.4 请求体、100 Continue 和已完成语义
-------------------------------------

当请求表明有请求体时，服务应当提供使用 owin.RequestBody
键访问请求体流。如果期望没有请求体数据 Stream.Null
可以作为占位符被使用。

当请求 Expect 头指明客户端请求 100 Continue,100 Continue
值由服务（决定是否）提供，应用程序禁止设置 owin.ResponseStatusCode
的值为100，100 Continue
仅只用于中间响应，使用它将会阻止应用程序提供最终响应（比如：200
OK）。在发生应用程序在请求体数据达到前读取请求体流情况，服务应当代表应用程序发送
100 Continue 。

（1）应用程序委托在请求完成前不应当结束请求并返回
任务，并把请求控制交给服务。一旦 AppFunc
完成应用程序不应当继续从请求流中读取数据。

（2）应用程序必须通过完成其返回的任务或抛出异常来发出响应主体完成或失败的信号。
在任务完成后，应用程序不应当向响应流中写任何数据。

（3）如果在应用程序委托执行期间服务发送 owin.CallCancelled
调用取消令牌，应用程序不应当尝试再从请求流读取数据，而应当迅速的完成应用程序

委托任务。

（4）应用程序不应当关闭或释放给定的（请求）流除非它完成了请求内容的使用。请求流的拥有者（比如服务或中间件）必须在应用程序委托任务完成时做必要的清理（工作）。

（5）任何从请求体流中抛出的异常【】都是致命的，而是应当通过在 AppFunc
中（同步）抛出（异常）或使用给定的异常致使异步任务失败来实现。

3.5 响应体
----------

服务在初始化环境字典是提供 owin.ResponseBody
键访问的请求体流。响应头、状态、说明词组等在第一次写入响应体前都是可以修改的。

在第一次写入时，服务验证和方式响应头，应用程序可以选择缓冲响应数据来延迟响应头确定。

（1）应用程序必须通过完成其返回的任务或抛出异常来发出响应主体完成或失败的信号。在任务完成后，应用程序不应当向响应流中写任何数据。

（2）如果在应用程序委托执行期间服务发送 owin.CallCancelled
调用取消令牌，应用程序不应当尝试再从请求流读取数据，而应当迅速的完成应用程序委托任务。

（3）应用程序不应当关闭或释放给定的（请求）流除非它完成了请求内容的使用。请求流的拥有者（比如服务或中间件）必须在应用程序委托任务完成时做必要的清理（工作）。应用程序不应当假定给定的流支持多个未完成的异步写操作。应用开发者应当在尝试使用这种方式前验证服务和所以在使用的中间件支持这种模式。

3.6 请求生命周期
----------------

请求完整范围或生命周期受到一些因素的限制，包含客户端、服务和应用程序委托。在最简单的场景中，一个请求生命周期结束是在应用程序委托完成和服务正常的结束请求。任何级别的错误可能导致请求过早的结束，或者可能在内部处理和允许请求继续（执行）。

owin.CallCancelled
键与取消调用令牌关联，当请求(需要)终止时服务使用其作为（终止请求）信号。如果在
AppFunc 任务完成前请求出现错误

这个（信号）应当被触发。也应当在提供商决定的任何（时间）点触发。中间件可以使用自己的（令牌）来替换它从而提供额外的粒度或功能，但他们应该把新令牌链到原始令牌上。

4 应用程序启动
==============

当宿主进程启动时，它通过一系列的步骤来设置应用程序。

1. 宿主创建属性 IDictionary\<string,
object\>，并且填充属性中所有启动数据或功能。

2. 宿主选择将被使用的服务，并且提供将属性集合提供给它，所以宿主可以近似的宣称拥有任意的功能。

3. 宿主定位应用程序启动类，并且提供属性集合来调用启动程序。

4. 应用程序读取或者设置属性集合中的配置，构造想要的请求处理管道，并且返回应用程序委托结果【返回】。

5. 宿主使用给定的应用程序委托和属性集合调用服务启动类，服务自己完成配置，开始接受请求，并且调用应用程序委托来处理这些请求。属性字典被设计为支持宿主、服务、中间件或应用程序用于读取或设置任何配置参数。

（1）启动属性字典必须非空、可修改和必须包含下表要求的键列表

（2）键比较必须使用 StringComparer.Ordinal

（3）键对应的值必须非空，除非另有说明

除了这些键，宿主、服务、中间件和应用程序等可以向属性字典添加任意与应用程序配置有关的数据。

添加键的准则和通用键定义列表可以在 CommonKeys addendum 规范中找到。

5.URI 重建
==========

应用程序通常需要重建请求的完整 URI 的能力，这个过程不可能完美，因为 HTTP
客户端通常

发送不完整的 URI 请求，但是 OWIN 为构建近似于（完整） URI
意图提供了选择。

5.1 URI 方案
------------

HTTP 客户端通常不会发送 URI 方案信息，并且依靠网络配置，OWIN
服务也许不能推断出它的正确值。

在这些情形下，服务也许必须手动的配置或计算出一个值。

（1）服务必须为 owin.RequestScheme 提供一个最佳的猜测值

5.2 主机名称
------------

在 HTTP/1.1
请求的上下文中，客户端发出请求的服务器的名称通常在请求的主机头字段值中表明，尽管它可能使用绝对请求
URI 来指定（详见 2616, sections 5.1.2, 19.6.1.1）。

【译者注：示例：GET http://www.w3.org/pub/WWW/TheProject.html HTTP/1.1】

服务必须在请求头字典中为 Host 键提供一个值，值的格式必须是
\<hostname\>\[:\<port\>\]。

这个值应当被宿主通过以下步骤来获取：

1\. 如果传入请求的 URL 是绝对 URL，则 Host 键对应的值必须是绝对 URL 中
Host 部分。

2\. 如果传入请求的 URL 不是绝对 URL，则 Host 键对应的值必须采用传入请求头
Host 字段的值。

3\. 如果传入请求的 Host 字段没有值，或者它的值是空白，则服务必须为 Host
键对应的值提供合理的最佳猜测值。

5.3 路径
--------

服务需要具有将应用程序委托映射到一些基础路径的能力。比如，服务可能有应用程序委托设置响应以"/my-app"开头的请求，在这种情形下环境字典中"owin.RequestPathBase"的值应当设置为"/my-app"。如果这个服务收到"/my-app/foo"请求，则环境字典中"owin.RequestPath"的值应当设置为"/foo"。

（1）环境字典中"owin.RequestPathBase"的值禁止使用斜杆结尾，并且必须以斜杆开始或者值是
String.Empty

（2）环境字典中"owin.RequestPath"的值必须使用斜杆开始，或者当"owin.RequestPathBase"不是
String.Empty 时，它可以是 String.Empty

5.4 URI 重构算法
----------------

下面的算法可以被用于构建近似完整请求 URI：

var uri =

(string)Environment\[\"owin.RequestScheme\"\] +

\"://\" +

Headers\[\"Host\"\].First() +

(string)Environment\[\"owin.RequestPathBase\"\] +

(string)Environment\[\"owin.RequestPath\"\];

if(Environment\[\"owin.RequestQueryString\"\] != \"\")

{

uri += \"?\" + (string)Environment\[\"owin.RequestQueryString\"\];

}

上面的结果可能与客户端发送请求使用的 URI
不相同；比如，服务可能进行一些重写来规范请求。

此外，这个主题在 URI Scheme and Hostname sections 中有附加说明。

5.5 百分比编码
--------------

URI 使用百分比编码来传输被允许使用之外的字符。百分比编码是被用于 URI
组件中的8位编码，这8位编码通过

UTF-8编码产生。大多数的 Web 服务器会在请求路径中实现百分比编码（see: RFC
2616 section 5.1.2, also 3.2.3），并且 OWIN 遵从这个默认规则。在 OWIN
中请求查询字符串以百分比编码形式呈现。一个百分比编码查询字符串可以包含'？'或'='字符，这将导致查询字符串不可理解。

（1）服务必须为\"owin.RequestPath\" 和 \"owin.RequestPathBase\"
的值提供百分比编码后的值

（2）服务必须为 \"owin.RequestQueryString\" 的值提供百分比编码后的值

6 错误处理
==========

尽管这儿有些标准的异常（比如： ArgumentException 和 IOException），
可能在正常的请求处理场景中可以预见发生。

但仅只处理这些异常对于创建健壮的服务或应用程序是不够的。如果服务希望健壮，它应该一致的处理所有类型的异常，这些异常由应用程序委托或报文体委托抛出或返回。处理机制（比如写日志、崩溃、重启和内部处理等）取决于服务和宿主进程。

6.1 应用程序错误
----------------

应用程序可能在以下位置产生异常：

（1）应用程序委托执行中抛出的异常

（2）由应用程序委托的返回任务提供

应用程序应当尝试处理内部异常，产生一个合适的响应（可能是 500
级别）而不是把异常传递给服务。

在应用程序提供响应后，服务应当在向底层协议写响应头前至少接收到一个向响应（体）流的写。提供这种方式，如果服务获取到应用程序委托任务的异常替代（原本）流的写入。服务仍能产生一个
500 级别的响应。

如果服务得到第一个向流的写操作，它可以安全的假定应用程序在内部就可能的处理了很多的异常，服务可以开始发送响应。

如果后续接收到异常，服务将视情况处理它。

6.2 服务错误
------------

如果服务在请求生命周期中发生错误，它应该向 owin.CallCancelled
提供取消调用令牌。

服务可以任何必要行为来终止请求，但是它应该包容应用程序委托完成的延迟。

7 版本说明
==========
