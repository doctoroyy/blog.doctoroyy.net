一、什么是跨域？
先来了解2个概念
1. 浏览器的同源策略
出于安全的考虑，同源策略控制了从一个源加载的文档或脚本如何与来自另一个源的资源进行交互，例如使用XMLHttpRequest时就会受到同源策略的约束。

2. 同源的定义
只有当两个站点的协议，主机，端口号全部相同才算是同源，下表是同源策略的检测示例：

URL	结果	原因
http://store.company.com/dir2/other.html	成功	只有路径不同
http://store.company.com/dir/inner/another.html	成功	只有路径不同
https://store.company.com/secure.html	失败	不同协议(https和http)
http://store.company.com:81/dir/etc.html	失败	不同端口 ( http:// 80是默认的)
http://news.company.com/dir/other.html	失败	不同域名 ( news和store )
只要这两个站点(协议, 主机, 端口号)有一个不同，那么就是跨域。

二、实现跨域的几种办法
1. CORS (跨域资源共享)
简单来说，跨域资源共享就是浏览器与服务器约定好的一种数据传输机制，当浏览器发出跨域请求时，会携带一个origin的header字段，这个字段的值对应当前页面location对象的origin值，浏览器发出这个请求，如果服务端返回的响应头部 中包含Access-Control-Allow-Origin这个字段且它的值包含origin或为*，那么浏览器就认为这是一个正确的响应，否则返回结果会被浏览器拦截。(其中*表示允许所有源)

这种方法需要服务端的配合，即在response headers加入Access-Control-Allow-Origin这个字段。

2. Nginx反向代理
一个最简单的Nginx反向代理示例可以是这样：

# 假设我们的主站点 book.doctoroyy.net 要从 api.doctoroyy.net 拿数据，
# 那么找到主站点的 nginx 配置文件，加入这些配置
# book.doctoroyy.net.conf
upstream remote_server {
  server api.doctoroyy.net;
}

location /api {
  proxy_pass https://remote_server;
}
那么此时 https://book.doctoroyy.net/api/ 就完全代理了 http://api.doctoroyy.net/ ,

假设原本想要发送请求到 http://api.doctoroyy.net/api/getAll ,

现在只需要发送到主站点的/api子路径下：https://www.doctoroyy.net/api/getAll

如此一来就巧妙的避开了浏览器的同源限制。

3. JSONP (hack手段)
前面提到，在浏览器的同源策略约束下，一个站点的文档或脚本不能随便请求另一个站点的数据，但是有一些例外情况：

<link>、<script>这些标签允许嵌入跨域脚本

一个JSONP的实现大概是这样：

在浏览器(客户)端:

<script>
  function initParams(json) {
    var data = json;
    console.log(data);
  }
</script>

<script src='https://api.doctoroyy.net/book?callback=initParams'></script>
在服务端：

data = {
 'key1': value1,
 'key2': value2,
 'key3': value3,
}

func = request.GET['callback']

return HttpResponse(func + '(' + json.dumps(data) + ')');
这样，其实就是相当于远程加载了一段JavaScript脚本，这个脚本的内容是调用本地initParams函数，初始化需要的数据。

4. 其他
Node 代理
具体实现大致就是服务端开启一个node服务，node服务拦截到来自当前页面的请求后，将请求分发给具体的服务器，前面提到，同源策略是针对浏览器的，服务端并不受这个限制。node将拦截到的请求做解析之后请求真正的服务器，得到数据后返回给客户端。

这种方式我没有具体实现过，但是如果结合nginx大致配置应该是这样：

nginx配置

# book.doctoroyy.net.conf

upstream node1 {
  server 127.0.0.1:8000; # 服务端启动的node服务
}

location /api {
  proxy_pass https://node1;
}

location /static {
  # 像一些静态资源的请求并不需要转发到node服务
}
node服务

# 假设拦截到请求 https://book.doctoroyy.net/api/getAll
# 向 https://api.doctoroyy.net/getAll 发送请求
# response = 来自 https://api.doctoroyy.net/getAll 的返回结果
# return response;
ps: 此外还有基于WebSocket、postMessage的实现，这里不做过多讨论。

三、参考来源
https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
