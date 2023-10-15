# 什么是模板
简单来说，就是网站内容的动态部分，如果有一个网站的内容几乎相同，但只有某些部分发生改变，那么他们很有可能使用了模板
模板看起来如下：
```
hello{user.name}
他们有一个静态罐和一个动态罐，hello为静态罐，{}里的内容为动态罐
```
这就是为什么编译器如何是知道该部分是动态的，另外需要注意的是，并非是所有模板看起来都是像上面那样，列如：
```
hello{{user.name}}
```
这是一个名为jinja模板引擎的示例，用流行的python框架（如django和flask）所使用
# 什么是模板注入
如题所示，它就是存在于web应用中的注入漏洞，例如：
```
template = "Bio: {{ user.bio }}"
render(template)
```
如果使用此模板显示用户的输入，那么它是完全安全的，因为我们所做的只是从数据库中获取当前用户的信息，然后将其返回给用户，但下面这个例子就不太一样了：
```
template = "Bio:  " + USER_INPUT
render(template)
```
如果用户的输入的成为模板的一部分，那么我们就有一个大问题，因为模板有能力执行任意代码，所以用户可以在服务器上获得一个shell
这种漏洞通常被称为服务器模板注入，攻击者可以在其中注入恶意模板代码来获得shell，但需要注意的一点是，她不仅限于服务器，只要模板可用，漏洞就可以存在于任何地方
# 实战
实战题目为：[护网杯 2018]easy_tornado
进入网站首页，发现了三个提示
![在这里插入图片描述](https://img-blog.csdnimg.cn/28d7a9df63824d6d9d881bcb0087491c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
一个是flag在/fllllllllllllag文件里
![在这里插入图片描述](https://img-blog.csdnimg.cn/d827e371f6f94cb4a5312d968827afcd.png)
一个是render这个python函数，作用是通过传递不同的参数形成不同的改变
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef382b787eff45338d229e9d5a9956e8.png)
一个是我们要如何访问flag的提示
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d5813ec6e3843aa8607cf5c1e64cd2f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
```
md5(cookie_secret+md5(filename))
```
意思是需要filehash这个GET参数需要filename（也就是/fllllllllllllag）与cookie_secret的md5加密才能访问最终的flag值
我们要想办法获取cookie_secret的值
我们先把/fllllllllllllag输入进去试试
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ddaa70c15684e578f31f4065506922c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0960829820dc4fce9552354a2e9d1e3a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
发现提示?msg=Error，我们修改Error这个字符串试试
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f15a5b5a93a44a5ada7ccaabe84671d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)发现在后面跟上{{}}也能用，这就是服务端模板注入攻击
![在这里插入图片描述](https://img-blog.csdnimg.cn/42b6357ff4424159900c36f58fb5bf17.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
然后我们需要想办法获得cookie_secret的值，题目是easy_tornado，tornado是一个框架，参考官方文档，发现存在附属文件handler.settings
```
官方文档：https://www.tornadoweb.org/en/latest/guide/templates.html#template-syntax
https://www.cnblogs.com/bwangel23/p/4858870.html
```
我们访问这个文件看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/134f73f5482d416bb730df0c12c4c15e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
成功的出现了我们的cookie_secret的值，接下来只需要按照之前的提示操作即可
```
md5(cookie_secret+md5(filename))
```
首先是对flag文件名/fllllllllllllag的md5加密，然后将cookie_secret与加密后的flag的MD5值再进行一次加密

![在这里插入图片描述](https://img-blog.csdnimg.cn/25e79eebbe924b5496a271ec43155562.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/495e84c8ee2f49758bd6847b9adea610.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
加密完成后访问网站
![在这里插入图片描述](https://img-blog.csdnimg.cn/f18aff8604ad4890acace4b18a797ee9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmExX01hMA==,size_20,color_FFFFFF,t_70,g_se,x_16)
得到flag
# 总结
在模板中不安全地嵌入用户输入会导致服务器端模板注入，这是一个非常容易被误认为是跨站点脚本(XSS) 或完全遗漏的严重漏洞。与 XSS 不同，模板注入可用于直接攻击 Web 服务器的内部结构，并经常获得远程代码执行 (RCE)，将每个易受攻击的应用程序变成潜在的支点。
有什么不懂的可以加我qq：3316735898



