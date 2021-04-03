### 1. 什么是XSS攻击

XSS是指恶意攻击者**利用**网站没有对用户提交数据进行转义处理或者过滤不足的**缺点**，进而添加一些代码，嵌入到web页面中去。使别的用户访问都会执行相应的嵌入代码。

### 2. XSS类型

存储型、DOM型、反射型

### 3. 造成XSS的原因

过于信任客户端提交的数据。攻击者利用网站对客户端提交数据的信任，在数据中插入一些符号以及JS代码，这些数据会成为应用代码的一部分，也就是说插入的一些攻击代码会被执行。

例子：

1. 比如在论坛中，攻击者在留言的input字段中填写`<script>alert(‘foolish!’)</script>`并提交留言，这样当其它用户访问论坛时查看留言就会获取到这段攻击代码并在网页执行。

2. Dom型攻击

` http://www.vulnerable.site/welcome.html?name=<script>alert(document.cookie)</script>`

受害者的浏览器接收到这个链接，发送HTTP请求到www.vulnerable.site并且接受到上面的HTML页。受害者的浏览器开始解析这个HTML为DOM，DOM包含一个对象叫document，document里面有个URL属性，这个属性里填充着当前页面的URL。当解析器到达javascript代码，它会执行它并且修改你的HTML页面。倘若代码中引用了document.URL，那么，这部分字符串将会在解析时嵌入到HTML中，然后立即解析，同时，javascript代码会找到(alert(…))并且在同一个页面执行它，这就产生了xss的条件。

### 4. 预防XSS

不相信用户提交的数据，对用户提交的信息进行过滤。

1、将重要的cookie标记为http only, 这样的话Javascript 中的document.cookie语句就不能获取到cookie了。

2、表单数据规定值的类型，例如：年龄应为只能为int、name只能为字母数字组合。。。。

4、对数据进行Html Encode 处理

5、过滤或移除特殊的Html标签， 例如: <script>, <iframe> , &lt; for <, &gt; for >, &quot for

6、过滤JavaScript 事件的标签。例如 "onclick=", "onfocus" 等等。



