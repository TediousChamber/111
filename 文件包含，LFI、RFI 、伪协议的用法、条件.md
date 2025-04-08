# 文件包含，LFI、RFI 、伪协议的用法、条件

## 文件包含

- 什么是包含，程序员吧一些经常重复利用的函数写到单文件中，在使用一些函数时无须再次编写，这种过程就叫**包含**

- **文件包含漏洞的形成原因**：由于网站功能需求，会让前端用户选择要包含的文件，而开发人员又没有对要包含的文件进行安全考虑，就导致攻击者可以通过修改文件的位置来让后台执行任意文件，从而导致文件包含漏洞

- **文件包含漏洞**是一种注入行漏洞，本质就是输入一段用户能够控制的脚本或者代码，并让服务端执行

- 文件包含漏洞分为**本地文件包含**(Loacl File Inclusion,LFI)和**远程文件包含(**Remote File Inclusion,RFI)。

- 常见的文件包含函数

  include（）找不到被包含的文件只会产生警告，脚本将继续执行。

  include_once（）与include（）类似，唯一区别是如果该文件中的代码已经被包含，则不会再次包含

  require（）找不到被包含的文件时会产生致命错误，并停止脚本执行

  require_once（)与require（）类似，唯一区别是如果该文件中的代码已经被包含，则不会再次包含
  
  若文件内容符合PHP语法规范，不管拓展名是什么都会被php解析
  
  ```php
  <?php
  	include $_GET['test'];
  ?>
  
  ```
  
  然后在创建一个phpinfo.php文件
  
  ```php
  <?php
  	phpinfo();
  ?>
  
  ```
  
  构造payload：?test=phpinfo.php
  
  ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89e6cfe24d494cec8c22d5caeb9a58b6.png)
  
  然后将phpinfo.php文件后缀改为txt，等都可以解析
  
  
  
  
  
  
  
  若文件内容不符合PHP语法规范则会暴漏其源码
  
  file_get_contents()
  highlight_file()
  fopen()
  readfile()
  fread()
  fgetss()
  fgets()
  parse_ini_file()
  show_source()
  file()
  var_dump(scandir('/'));

## 本地包含

- 本地包含：是指被包含的文件位于服务器本地，不论是本地包含还是远程包含，都依赖于php.ini中的两个配置项

  ```
  allow_url_fopen （默认开启）
  allow_url_include #（默认关闭，远程文件包含必须开启）
  ```



- 可以通过下面的代码

```php
<?php
	$file=$_GET['filename'];
	include($file);
?>

```

1. 可以通过使用绝对路径直接读取。例如通过构造payload：filename=C:\Windows\system.ini
2. 使用./表示当前位置路径，.../表示上一级路径的位置，在linux中同样适用，例如构造路径../../windows/system.ini

- 一些常见的敏感目录信息路径

  windows系统：C:\boot.ini //查看系统版本
  C:\windows\system32\inetsrv\MetaBase.xml //IIS配置文件
  C:\windows\repair\sam //存储Windows系统初次安装的密码
  C:\ProgramFiles\mysql\my.ini //Mysql配置
  C:\ProgramFiles\mysql\data\mysql\user.MYD //MySQL root密码
  C:\windows\php.ini //php配置信息

  linux/unix系统：/etc/password //账户信息
  /etc/shadow //账户密码信息
  /usr/local/app/apache2/conf/httpd.conf //Apache2默认配置文件
  /usr/local/app/apache2/conf/extra/httpd-vhost.conf //虚拟网站配置
  /usr/local/app/php5/lib/php.ini //PHP相关配置
  /etc/httpd/conf/httpd.conf //Apache配置文件
  /etc/my.conf //mysql配置文件

  

## 远程文件包含

- **远程文件包含**就是允许攻击者包含一个远程的文件,一般是在远程服务器上预先设置好的脚本。 此漏洞是因为浏览器对用户的输入没有进行检查，导致不同程度的信息泄露、拒绝服务攻击 甚至在目标服务器上执行代码。
- **远程服务器**通常位于离用户所在地点较远的地方，可以是不同的国家、城市甚至洲际之间。用户通过网络连接，不受地理位置的限制，即可访问远程服务器上的资源。

​         

```php
<?php
	$path=$_GET['path'];
	include($path . '/phpinfo.php');
?>

```

访问本地site目录下的phpinfo.php文件

![img](https://i-blog.csdnimg.cn/blog_migrate/a4873ed95f09f9cdf0a405779346c625.png)

该页面并没有对$path做任何过滤，因此存在文件包含漏洞

我们可以在远程web服务器/site/目录下创建一个test.php，内容为phpinfo（）,利用漏洞去读取文件

但是代码会在我们输入的路径后面加上'/phpinfo.php'后缀，可以使用？号截断，在路径后面输入？号，服务器会认为？号后面的内容为GET方法传递的参数

## 本地与远程文件区别

1. **文件来源**：
   - **LFI**：被包含的文件位于本地服务器上。
   - **RFI**：被包含的文件位于远程服务器上。
2. **攻击方式**：
   - **LFI**：攻击者需要知道目标服务器上文件的具体路径，才能通过文件包含漏洞读取或执行该文件。
   - **RFI**：攻击者只需将恶意代码上传到远程服务器，并通过文件包含漏洞包含该远程文件，即可在目标服务器上执行任意代码。
3. **危害程度**：
   - **LFI**：可能导致敏感信息泄露、文件篡改等危害，但通常不会直接导致远程代码执行（除非被包含的文件本身包含可执行代码）。
   - **RFI**：可能导致严重的远程代码执行漏洞，攻击者可以在目标服务器上执行任意代码，从而完全控制目标服务器。
4. **配置要求**：
   - **LFI**：通常不需要特殊的配置要求，只要应用程序存在文件包含漏洞即可。
   - **RFI**：需要服务器配置允许通过URL进行文件包含（例如，PHP中需要设置`allow_url_include = on`）。然而，由于RFI漏洞的严重危害，现代服务器配置通常默认禁用此选项。

总结来看，远程文件包含和本地文件包含的主要区别在于被包含文件的位置和来源上。由于RFI漏洞的严重危害，开发者需要特别注意防范此类漏洞，确保应用程序的安全性。

## php伪协议用法，条件

### php：//filter

php://filter 是一种元封装器， 设计用于数据流打开时的筛选过滤应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() 和 file_get_contents()， 在数据流内容读取之前没有机会应用其他过滤器。
例子***php://filter/read=convert.base64encode/resource=index.php***
***php://filter/resource=index.php***

php://filter 伪协议组成：
read=<读链的筛选列表>
resource=<要过滤的数据流>
write=<写链的筛选列表>
php://filter/read=处理方式（base64编码，rot13等等）/resource=要读取的文件

read=convert.base64-encode是将resource指向的内容进行转化,将其变成base64编码,并输出结果.这样处理后的输出不再是一个可执行的 PHP 文件，而是一个 Base64 编码的字符串。因此,可以直接被include_once给输出来

read 对应要设置的过滤器：
常见的过滤器分字符串过滤器、转换过滤器、压缩过滤器、加密过滤器
其中convert.base64-encode ，convert.base64-decode都属于 转换过滤器



### php://stdin, php://stdout 和 php://stderr

php://stdin、php://stdout 和 php://stderr 允许直接访问 PHP 进程相应的输入或者输出流。 数据流引用了复制的文件描述符，所以如果你打开 php://stdin 并在之后关了它， 仅是关闭了复制品，真正被引用的 STDIN 并不受影响。 推荐你简单使用常量 STDIN、 STDOUT 和 STDERR 来代替手工打开这些封装器

php://stdin 是只读的， php://stdout 和 php://stderr 是只写的。

### php://input 

php://input 是个可以访问请求的原始数据的只读流。 如果启用了 [enable_post_data_reading](https://www.php.net/manual/zh/ini.core.php#ini.enable-post-data-reading) 选项， php://input 在使用 `enctype="multipart/form-data"` 的 POST 请求中不可用

### php://output

php://output 是一个只写的数据流， 允许你以 [print](https://www.php.net/manual/zh/function.print.php) 和 [echo](https://www.php.net/manual/zh/function.echo.php) 一样的方式 写入到输出缓冲区。

### php://fd 

php://fd 允许直接访问指定的文件描述符。 例如 php://fd/3 引用了文件描述符 3

### php://memory 和 php://temp 

php://memory 和 php://temp 是一个类似文件 包装器的数据流，允许读写临时数据。 两者的一个区别是 php://memory 总是把数据储存在内存中， 而 php://temp 会在内存量达到预定义的限制后（默认是 2MB）存入临时文件中。 临时文件位置的决定和 sys_get_temp_dir() 的方式一致。

php://temp 的内存限制可通过添加 /maxmemory:NN 来控制，NN 是以字节为单位、保留在内存的最大数据量，超过则使用临时文件。

