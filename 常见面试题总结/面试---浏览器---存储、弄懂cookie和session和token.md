> 涉及的面试题：有几种方式可以实现存储功能，分别有什么优缺点？

有cookie、localStorage、sessionStorage、indexDB这几种存储方式。

![mxb0js.png](https://s2.ax1x.com/2019/08/31/mxb0js.png)

cookie是网站为了标识用户身份而存储在用户本地终端上的数据（通常经过加密），cookie数据始终存储在同源的HTTP请求中携带（即使不需要），即会在浏览器和服务器之间来回传递。

localStorage和sessionStorage不会自动把数据发送给服务器，仅在本地保存。

### cookie和session和token

**http是一个无状态协议**

什么是无状态呢？就是说这一次请求和上一次请求没有任何关系，互不认识，没有关联的，这样做的优点是速度快，但是在实际应用中我们常常需要把两次请求关联起来。

**cookie和session**

由于http的无状态性，为了使某个域名下的所有网页能够共享某些数据，session和cookie出现了。

两者的区别：

cookie是保存在客户端的，session是保存在服务端的。

cookie的缺点：1. 大小和数量都有限制；2. cookie是存在客户端的很可能会被删除和修改，是不安全的；3. cookie如果很大，每次请求都要带上，这样就会影响传输效率

session是基于cookie来实现的，但是session每次传输的不是数据而是代表客户端的一个唯一的ID，写在客户端的cookie中。

session的优点就是传输数据量小，比较安全。

session的缺点是如果不做特殊的处理比较容易失效、过期、丢失或者session过多导致服务器内存溢出。

现在要是用的话也会是cookie + session，一般不会单独使用。

**token**

token也称作令牌，由uid+time+sign+[固定参数]

- UID：用户唯一身份标识
- time：当前时间的时间戳
- sign：签名，使用hash压缩成定长的十六进制字符串，以防止第三方恶意拼接
- 固定参数（可选）：将一些常用的固定参数加入到token中是为了避免重复查库

token的认证方式类似于临时的证书签名，并且是一种服务端无状态的认证方式，所谓无状态就是服务端并不会保存身份认证的相关数据。

**存放**

token在客户端一般存放在localStorage、sessionStorage、cookie中。在服务器一般存放在数据库中。

当讨论token身份验证的时候，一般都是说JSON Web Tokens（JWT）。虽然有很多种方式可以实现token，但是JWT已经成为了事实上的标准，所以会将token和JWT混用。

**token相对于cookie的优势**

**无状态**

token的验证时无状态的，这也许是它相对于cookie来说最大的优点。后端服务不需要记录token。每个令牌都是独立的，包括检查其有效性所需要的数据，并通过声明传达用户信息。

服务器唯一的工作就是在用户登录成功的请求上签署token，并验证传入的token是否有效。

**token可以抵御跨站请求伪造（CSRF）而cookie+session不行**

举一个CSRF攻击的例子：

加入用户正在登录银行网页，同时登录了攻击者的网页，并且银行网页并没有对CSRF攻击进行保护，攻击者就可以在银行网页上面放一个表单，该表单提交src为http://www.bank.com/api/transfer，body为count=1000&to=Tom。

倘若是cookie+session，用户提交表单的时候已经转给Tom1000元了，因为form发起的post请求，并不会受浏览器的同源策略的限制，因此可以任意使用其他域的cookie向其他域发送post请求，形成CSRF攻击。在post请求的瞬间，cookie会被浏览器自动添加到请求头中。

token则不会，token是开发者专门为防止CSRF攻击而特别设计的令牌，浏览器不会自动将token添加到headers里，攻击者也无法访问用户的token，所以提交的表单无法通过服务器的验证，也就无法形成攻击。

总结：

- token不是防止XSS的，而是为了方式CSRF专门设计的
- CSRF攻击的原因是浏览器会自动带上cookie，而浏览器不会自动带上token

**分布式情况下的session和token**

我们知道session是有状态的，一般存在服务器的内存或硬盘中，当服务器采用分布式或集群的时候，session就会面对负载均衡的问题。

- 负载均衡多服务器的情况，不好确认当前用户是否登录，因为多服务器不共享session。

而token是无状态的，token字符串里就保存了所有的用户信息
- 客户端登录传递信息给服务器，服务器收到后把用户信息加密（token）传给客户端，客户端将token存在localStorage中。客户端每次访问都传递token，服务端解密token，就知道这个用户是谁了，通过CPU加解密，服务端就不需要存储session占用存储空间了，就很好的解决了多服务器负载均衡的问题了，这个方法叫JWT。

总结：

- session存储在服务器中，可以理解为一个状态列表，拥有一个唯一识别符号sessionid，通常存放于cookie中。服务器收到cookie后解析出sessionid，再去session列表中查找，才能找到相应的session，依赖于cookie。
- cookie类似一个令牌，装有sessionid，存储在客户端，浏览器通常会自动添加
- token也类似一个令牌，无状态，用户信息都被加密到token中，服务器收到token后解密就知道是哪个用户了。需要开发者手动添加。