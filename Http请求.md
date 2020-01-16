## CORS（跨域资源访问）机制

1、CORS需要服务端和客户端共同支持。

2、客户端（浏览器）进行CORS的过程是自动进行的，用户不会察觉到

3、服务端需要**实现CORS接口**

4、浏览器将CORS请求分为两类：简单请求和非简单请求，对于简单请求来说，需要同时满足以下条件

1. 请求的方法是HEAD|GET|POST

2. HTTP头信息字段为以下字段的子集

   - Accept
   - Accept-language
   - Content-language
   - Last-Event-ID
   - Content-Type:application/x-www-form-urlencoded或multipart/form-data或text/plain

   凡是不能同时满足以上两个条件的请求，都是非简单请求

5、**简单请求的处理方法：**在请求时，浏览器会自动在请求头中添加一个Origin字段，用于向非同源的后端服务表示本请求来自于哪个源。对于这个请求，如果后端服务的配置中发现该源不在允许范围之内，服务器会正常返回响应，响应消息的响应头中没有Access-Control-Allow-Origin字段。这时浏览器就知道，服务端不允许本源的请求。错误信息会被XMLHttpRequest的onerror回调函数捕获。

如果后端服务允许该源的请求，则会在响应消息的响应头中添加以下三个字段：

- Access-Control-Allow-Origin：这个字段的值要么是浏览器发起请求时的Origin字段的值，要么是*，表示允许任意域名的请求。

- Access-Control-Allow-Credentials：这个字段是可选的，值为true或者false。表示是否允许浏览通过cookie携带凭证。

- Access-Control-Expose-Headers：这个字段是可选的，这个字段的值表示允许浏览器获取header中的某个被暴露的字段。

  如果客户端想通过cookie发送凭证消息，除了服务端Access-Control-Allow-Credentials值为true之外，在客户发发起请求的时候还需要设置ajax的withCredentials = true，同时Access-Control-Allow-Origin也必须为客户端的源，不能为*。

6、**非简单请求的处理方法：**比如DELETE、PUT，或者是Content-type类型为application/json的请求。这种请求在和非同源服务器通信时，客户端会提前多发起一次OPTIONS请求，称为预检请求。目的是先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。比如，浏览器发起这样的请求：

```javascript
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

浏览器会在PUT请求执行前构造这样的一个请求发送至服务器：

> OPTIONS /cors HTTP/1.1
> Origin: http://api.bob.com
> Access-Control-Request-Method: PUT
> Access-Control-Request-Headers: X-Custom-Header
> Host: api.alice.com
> Accept-Language: en-US
> Connection: keep-alive
> User-Agent: Mozilla/5.0...

Access-Control-Request-Method：用于指定客户端发起的请求是何种类型

Access-Control-Request-Headers: X-Custom-Header：用于说明客户端会额外发送的自定义请求头字段。

如果服务器同意请求，则会返回类似这样响应：

> HTTP/1.1 200 OK
> Date: Mon, 01 Dec 2008 01:15:39 GMT
> Server: Apache/2.0.61 (Unix)
> Access-Control-Allow-Origin: http://api.bob.com
> Access-Control-Allow-Methods: GET, POST, PUT
> Access-Control-Allow-Headers: X-Custom-Header
>
> Access-Control-Allow-Credentials: true
> Access-Control-Max-Age: 1728000
>
> Content-Type: text/html; charset=utf-8
> Content-Encoding: gzip
> Content-Length: 0
> Keep-Alive: timeout=2, max=100
> Connection: Keep-Alive
> Content-Type: text/plain

响应说明了服务器允许的域名、请求方法、自定的请求头。

Access-Control-Allow-Origin字段同样可能为*，意义与处简单请求时一样。

Access-Control-Max-Age字段可选，表示在这个时间内不用再发起OPTIONS请求。